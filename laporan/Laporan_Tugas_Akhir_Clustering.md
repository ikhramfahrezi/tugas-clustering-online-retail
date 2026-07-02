# Laporan Tugas Akhir Data Mining
## Preprocessing dan Unsupervised Learning (Clustering): Segmentasi Pelanggan Online Retail

**Nama:** Ikhram Fahrezi
**NIM:** 241011402337
**Kelas:** 04TPLM008
**Mata Kuliah:** Data Mining
**Topik:** Customer Segmentation menggunakan K-Means Clustering

---

## 1. Pemilihan Dataset

Dataset yang digunakan adalah **Online Retail Dataset** dari **UCI Machine Learning Repository**
(sumber: https://archive.ics.uci.edu/ml/datasets/online+retail).

Dataset ini berisi data transaksi nyata dari sebuah perusahaan ritel online (non-store) yang berbasis di
Inggris, dengan periode transaksi **1 Desember 2010 – 9 Desember 2011**, terdiri dari **541.909 baris**
dan **8 kolom**: `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`,
`CustomerID`, `Country`.

Dataset ini dipilih karena:
1. **Bersifat publik dan kredibel** (banyak digunakan dalam riset akademik, termasuk paper RFM
   customer segmentation karya Chen, Sain & Guo, 2012).
2. **Memiliki kecacatan data yang nyata** — bukan dataset yang sudah bersih — sehingga relevan untuk
   melatih tahap *cleaning* sebelum clustering.
3. **Cocok secara konseptual untuk clustering**, karena data transaksi dapat ditransformasi menjadi
   profil perilaku pelanggan (RFM) yang merupakan kasus klasik *unsupervised learning*.

## 2. Identifikasi Kecacatan Data (Sebelum Cleaning)

Hasil pemeriksaan awal pada data mentah menemukan kecacatan-kecacatan berikut:

| Jenis Kecacatan | Jumlah | Keterangan |
|---|---|---|
| Missing value `CustomerID` | 135.080 baris (24,93%) | Transaksi tanpa identitas pelanggan terdaftar |
| Missing value `Description` | 1.454 baris | Nama produk kosong |
| Baris duplikat (identik penuh) | 5.268 baris | Transaksi tercatat berulang |
| Format tidak konsisten | — | `InvoiceDate` bertipe teks bukan datetime; `CustomerID` bertipe float bukan integer; ditemukan karakter BOM tersembunyi pada nama kolom pertama saat dibaca dengan encoding standar |
| Transaksi pembatalan (`InvoiceNo` berawalan `'C'`) | 9.288 baris | Order yang dibatalkan, bukan transaksi riil |
| `Quantity` negatif | 10.624 baris | Retur barang / kesalahan input |
| `UnitPrice` ≤ 0 | 2.517 baris | Data tidak valid secara bisnis |

## 3. Tahapan Cleaning Data

Proses cleaning dilakukan secara berurutan dan sistematis sebagai berikut:

### 3.1 Penanganan Missing Values
- Baris dengan `CustomerID` kosong **dihapus**. Keputusan ini diambil karena tujuan akhir analisis adalah
  menyegmentasi *pelanggan* — baris tanpa identitas pelanggan tidak dapat dipetakan ke profil RFM siapa pun,
  sehingga penghapusan lebih tepat dibanding imputasi (mengisi `CustomerID` palsu akan merusak makna data).
- Nilai `Description` yang kosong **diimputasi** dengan label `'UNKNOWN ITEM'`, karena kolom ini hanya bersifat
  deskriptif dan tidak digunakan dalam perhitungan numerik clustering, sehingga aman untuk diisi placeholder
  alih-alih dihapus barisnya.

### 3.2 Penanganan Data Duplikat
Baris yang identik di seluruh kolom dihapus menggunakan `drop_duplicates()`, karena duplikasi akan
menggandakan kontribusi suatu transaksi pada perhitungan Frequency dan Monetary pelanggan, sehingga
bisa menggeser hasil segmentasi secara tidak adil.

### 3.3 Standarisasi Format Data
- `InvoiceDate` dikonversi dari string ke tipe `datetime` agar dapat dihitung selisih harinya (untuk Recency).
- `CustomerID` dikonversi dari `float` ke `integer` karena merepresentasikan ID, bukan nilai desimal.
- Spasi berlebih pada `InvoiceNo` dan `Country` dirapikan (`strip()`).
- File dibaca dengan encoding `utf-8-sig` untuk menangani karakter BOM tersembunyi yang sebelumnya
  menyebabkan nama kolom `InvoiceNo` tidak terbaca dengan benar.

### 3.4 Penanganan Data Tidak Valid
Transaksi pembatalan (`InvoiceNo` berawalan `'C'`), serta baris dengan `Quantity` negatif atau
`UnitPrice` ≤ 0 dihapus, karena bukan merepresentasikan transaksi pembelian riil dan akan
mendistorsi perhitungan total belanja (Monetary) pelanggan jika diikutkan.

**Hasil akhir setelah cleaning:** dari 541.909 baris transaksi mentah, tersisa **392.692 baris**
transaksi yang valid dan bersih untuk dianalisis lebih lanjut (≈ 72,5% dari data asli).

## 4. Feature Engineering: Transformasi ke RFM

Karena data bersifat transaksional (1 baris = 1 item dalam 1 invoice), data perlu ditransformasi ke
level **pelanggan** menggunakan model **RFM**:

- **Recency** — jumlah hari sejak transaksi terakhir pelanggan tersebut (dihitung relatif terhadap
  tanggal transaksi terakhir di seluruh dataset + 1 hari).
- **Frequency** — jumlah invoice (transaksi) unik yang pernah dilakukan pelanggan.
- **Monetary** — total nilai uang (`Quantity × UnitPrice`) yang pernah dibelanjakan pelanggan.

Hasil transformasi ini menghasilkan **4.338 baris pelanggan unik**, masing-masing dengan 3 fitur numerik
(Recency, Frequency, Monetary) yang menjadi dasar clustering.

### Penanganan Outlier
Distribusi `Frequency` dan `Monetary` sangat *skewed* — ada pelanggan wholesaler dengan total belanja
hingga ratusan ribu poundsterling, jauh di atas pelanggan retail biasa. Outlier ini ditangani dengan
**capping berbasis IQR** (nilai di atas *upper bound* dipotong ke *upper bound*, bukan dihapus), agar
pelanggan tersebut tetap terhitung dalam analisis namun tidak mendominasi jarak Euclidean saat clustering.

## 5. Scaling / Standarisasi

K-Means mengandalkan jarak Euclidean antar titik data, sehingga fitur dengan skala besar (`Monetary`,
dalam satuan poundsterling, bisa mencapai ribuan) akan mendominasi fitur dengan skala kecil (`Recency`,
dalam satuan hari) jika tidak distandarisasi. Seluruh fitur RFM distandarisasi menggunakan
**StandardScaler** (mean = 0, standar deviasi = 1) sebelum dimasukkan ke algoritma clustering.

## 6. Implementasi Clustering: K-Means

Algoritma yang digunakan adalah **K-Means Clustering**, karena:
- Efisien untuk dataset berukuran menengah seperti ini (~4.300 pelanggan).
- Hasilnya mudah diinterpretasikan secara bisnis melalui rata-rata fitur per cluster (centroid).
- Merupakan algoritma standar pada kasus RFM customer segmentation di industri.

### 6.1 Menentukan Jumlah Cluster (k)
Dua metode evaluasi digunakan untuk menentukan k optimal:

| k | Inertia (WCSS) | Silhouette Score |
|---|---|---|
| 2 | 6760.3 | **0.5387** (tertinggi) |
| 3 | 3580.0 | 0.5139 |
| **4** | **2492.2** | **0.4901** |
| 5 | 2085.2 | 0.4300 |
| 6 | 1707.8 | 0.4248 |
| ... | ... | ... |
| 10 | 1106.3 | 0.3563 |

Silhouette score tertinggi secara statistik murni berada pada **k=2**, namun jumlah cluster ini hanya
membedakan pelanggan "aktif" vs "tidak aktif" — terlalu sederhana dan kurang *actionable* untuk
menyusun strategi pemasaran yang berbeda-beda. Grafik Elbow Method menunjukkan penurunan inertia mulai
melandai signifikan di **k=4**, dan k=4 juga merupakan jumlah segmen standar dalam praktik RFM
segmentation (*Champions, Loyal, At Risk/Potential, Hibernating*), dengan silhouette score yang masih
tergolong baik (0.49, mendekati 0.5). Atas pertimbangan **trade-off antara skor statistik dan
interpretasi/kegunaan bisnis**, dipilih **k = 4** sebagai jumlah cluster final.

### 6.2 Hasil Model Final
Model K-Means dengan **k=4** (`random_state=42`, `n_init=10`) menghasilkan **Silhouette Score = 0.4901**,
yang menunjukkan pemisahan antar cluster cukup baik (jauh di atas 0, mendekati 0.5).

## 7. Visualisasi Hasil Clustering

Tiga jenis visualisasi dibuat untuk mengevaluasi dan menjelaskan hasil clustering secara visual:

1. **Elbow Method & Silhouette Score plot** — menunjukkan dasar pemilihan k=4 (lihat `elbow_silhouette.png`).
2. **Scatter plot 2D hasil PCA** — RFM yang berdimensi 3 diproyeksikan ke 2 dimensi utama menggunakan PCA,
   menunjukkan 4 cluster yang terpisah dengan cukup jelas dan rapi (lihat `cluster_pca_2d.png`).
3. **Scatter plot 3D** — menampilkan posisi setiap pelanggan langsung pada sumbu Recency, Frequency,
   dan Monetary asli, diwarnai berdasarkan cluster (lihat `cluster_rfm_3d.png`).
4. **Boxplot per fitur per cluster** — menunjukkan distribusi nilai Recency, Frequency, dan Monetary
   pada masing-masing cluster, untuk memverifikasi bahwa keempat cluster memang memiliki karakteristik
   yang berbeda secara statistik (lihat `cluster_boxplots.png`).

## 8. Profil dan Interpretasi Bisnis Tiap Cluster

| Cluster | Jumlah Pelanggan | Recency rata-rata (hari) | Frequency rata-rata | Monetary rata-rata (£) | Label Segmen |
|---|---|---|---|---|---|
| 0 | 340 (7,8%) | 16,5 | 14,3 | 5.113,3 |  **Champions** |
| 3 | 817 (18,8%) | 33,8 | 6,3 | 2.566,7 |  **Loyal Customers** |
| 1 | 2.168 (50,0%) | 51,6 | 2,2 | 634,4 |  **Regular / Potential Customers** |
| 2 | 1.013 (23,4%) | 252,9 | 1,5 | 428,5 |  **Hibernating / At Risk Customers** |

**Interpretasi & rekomendasi strategi per segmen:**

- **Champions (Cluster 0)** — Pelanggan paling baru aktif belanja, paling sering, dan nilai belanja
  paling besar. Ini adalah pelanggan paling bernilai bagi bisnis. **Strategi:** berikan program loyalitas
  eksklusif, akses produk baru lebih dulu, dan apresiasi khusus agar tetap setia.
- **Loyal Customers (Cluster 3)** — Pelanggan setia dengan frekuensi dan nilai belanja cukup tinggi,
  meski belum sebesar Champions. **Strategi:** dorong dengan program *upsell/cross-sell* agar naik kelas
  menjadi Champions.
- **Regular/Potential Customers (Cluster 1)** — Segmen terbesar (50% dari total pelanggan), dengan
  frekuensi dan nilai belanja menengah ke bawah. **Strategi:** kampanye *re-engagement* (diskon,
  rekomendasi produk personal) untuk meningkatkan frekuensi belanja mereka.
- **Hibernating/At Risk Customers (Cluster 2)** — Pelanggan yang sudah lama tidak belanja
  (rata-rata 252,9 hari/~8 bulan), dengan frekuensi dan nilai belanja paling rendah. **Strategi:**
  kampanye *win-back* (misalnya kupon khusus "kami merindukanmu") agar tidak benar-benar churn.

## 9. Kesimpulan

1. Dataset Online Retail dari UCI Machine Learning Repository memiliki kecacatan data yang nyata dan
   signifikan (missing values ~25%, duplikat, transaksi tidak valid, format tidak konsisten), yang
   berhasil ditangani melalui proses cleaning sistematis sehingga data siap dianalisis.
2. Transformasi data transaksional menjadi data level-pelanggan melalui pendekatan RFM (Recency,
   Frequency, Monetary) merupakan langkah feature engineering kunci yang membuat data cocok untuk
   *unsupervised learning*.
3. Algoritma **K-Means** dengan **k=4** (dipilih melalui kombinasi Elbow Method, Silhouette Score, dan
   pertimbangan interpretasi bisnis) berhasil menghasilkan 4 segmen pelanggan yang berbeda secara
   statistik (Silhouette Score = 0.49) dan jelas secara bisnis: **Champions, Loyal Customers,
   Regular/Potential Customers,** dan **Hibernating/At Risk Customers**.
4. Hasil segmentasi ini dapat langsung dimanfaatkan oleh tim marketing untuk menyusun strategi yang
   berbeda bagi setiap kelompok pelanggan, alih-alih memberikan promosi yang sama untuk seluruh
   pelanggan ("one-size-fits-all").

## 10. Daftar File yang Disertakan

| File | Keterangan |
|---|---|
| `Online_Retail_raw.csv` | Dataset asli/mentah (sebelum cleaning) |
| `Customer_Segmentation_Clustering.ipynb` | Source code lengkap (Jupyter Notebook), sudah dieksekusi dengan output |
| `Laporan_Tugas_Akhir_Clustering.md` | Dokumen laporan ini |
| `transactions_cleaned.csv` | Data transaksi setelah proses cleaning |
| `rfm_clustered.csv` | Tabel RFM per pelanggan beserta label cluster hasil akhir |
| `cluster_profile.csv` | Ringkasan profil (rata-rata RFM) tiap cluster |
| `elbow_silhouette.png`, `cluster_pca_2d.png`, `cluster_rfm_3d.png`, `cluster_boxplots.png` | Visualisasi hasil clustering |

---
*Sumber dataset: Chen, D., Sain, S.L., & Guo, K. (2012). Data mining for the online retail industry:
A case study of RFM model-based customer segmentation using data mining. Journal of Database Marketing
& Customer Strategy Management, 19(3), 197–208.*
