# Tugas Akhir Data Mining — Customer Segmentation (K-Means Clustering)

## 📁 Struktur Folder

```
tugas-clustering-online-retail/
├── data/
│   └── Online_Retail_raw.csv        
├── notebook/
│   └── Customer_Segmentation_Clustering.ipynb   
├── laporan/
│   └── Laporan_Tugas_Akhir_Clustering.md        
├── outputs/                          
├── requirements.txt                  
└── README.md                         
```

---

## ✅ STEP 0 — Cek yang harus sudah terinstall

1. **Python 3.10 atau lebih baru.**
   Cek dengan buka terminal/CMD, ketik:
   ```
   python --version
   ```
   Kalau belum ada / versinya jauh lebih lama, install dulu dari https://www.python.org/downloads/
   (saat install di Windows, **centang "Add Python to PATH"**).

2. **VSCode.**
   Kalau belum ada, download di https://code.visualstudio.com/

---

## ✅ STEP 1 — Buka folder project di VSCode

1. Extract/pindahkan folder `tugas-clustering-online-retail` ini ke lokasi yang mudah diingat,
   misalnya `D:\Tugas\DataMining\`.
2. Buka VSCode → `File` → `Open Folder...` → pilih folder `tugas-clustering-online-retail`.
3. Buka Terminal bawaan VSCode: menu `Terminal` → `New Terminal` (atau `Ctrl + ~` / `Ctrl + backtick`).
   Pastikan terminal ini posisinya sudah di dalam folder project (cek dengan `pwd` di Mac/Linux atau
   `cd` tanpa argumen di Windows — harus menunjukkan folder `tugas-clustering-online-retail`).

---

## ✅ STEP 2 — Install Extension VSCode yang dibutuhkan

Klik ikon **Extensions** di sidebar kiri (ikon kotak susun/4 kotak), lalu cari & install:

1. **Python** (publisher: Microsoft) — wajib.
2. **Jupyter** (publisher: Microsoft) — wajib, supaya bisa buka & jalankan file `.ipynb` langsung di VSCode.

Tunggu sampai kedua extension selesai terinstall (biasanya muncul tombol "Reload" — klik kalau diminta).

---

## ✅ STEP 3 — Buat Virtual Environment (venv)

Virtual environment ini supaya library tugas ini tidak bercampur dengan project Python lain di
komputer kamu. Di terminal VSCode (pastikan posisi masih di root folder project), jalankan:

**Windows:**
```
python -m venv venv
venv\Scripts\activate
```

**Mac/Linux:**
```
python3 -m venv venv
source venv/bin/activate
```

Setelah aktif, di awal baris terminal akan muncul tulisan `(venv)`. Kalau sudah muncul, lanjut ke step berikutnya.

> ⚠️ **Setiap kali kamu menutup dan membuka ulang VSCode**, kamu harus mengulangi perintah
> `activate` di atas sebelum kerja lagi (kecuali kamu set kernel Jupyter-nya, lihat Step 5 —
> biasanya VSCode otomatis mengaktifkan venv kalau kamu pilih kernel yang benar).

---

## ✅ STEP 4 — Install Semua Library yang Dibutuhkan

Masih di terminal yang sudah aktif `(venv)`, jalankan:

```
pip install -r requirements.txt
```

Tunggu sampai proses selesai (biasanya 1-3 menit tergantung koneksi internet). Kalau sukses, akan
muncul tulisan `Successfully installed ...` di baris terakhir.

**Kalau ada error saat install:** coba update pip dulu dengan `python -m pip install --upgrade pip`,
lalu ulangi perintah `pip install -r requirements.txt`.

---

## ✅ STEP 5 — Buka Notebook & Pilih Kernel yang Benar

1. Di sidebar kiri VSCode (Explorer), klik folder `notebook/` → buka file
   `Customer_Segmentation_Clustering.ipynb`.
2. Notebook akan terbuka dalam tampilan sel-sel (cell). Di **pojok kanan atas** akan ada tombol
   bertuliskan **"Select Kernel"**.
3. Klik tombol itu → pilih **"Python Environments..."** → pilih environment yang ada tulisan
   `venv` (path-nya akan mengarah ke folder `venv` yang baru kamu buat di Step 3).
4. Kalau venv tidak muncul di pilihan, klik ikon refresh/reload di pojok kanan atas, atau tutup
   dan buka ulang VSCode lalu ulangi langkah ini.

> 🔑 Ini langkah paling sering bikin error ("ModuleNotFoundError") kalau kelewat — pastikan kernel
> yang dipilih benar-benar dari `venv`, bukan Python sistem/global.

---

## ✅ STEP 6 — Jalankan Notebook Sel demi Sel

1. Klik sel paling atas (judul tugas).
2. Tekan **`Shift + Enter`** untuk menjalankan sel itu dan otomatis pindah ke sel berikutnya.
3. Ulangi `Shift + Enter` untuk **setiap sel, dari atas ke bawah, tanpa melompat**.
   - Sel **markdown** (teks penjelasan, background putih/abu) tidak perlu dijalankan tapi tetap
     boleh ditekan `Shift+Enter` saja untuk lewat ke sel berikutnya.
   - Sel **code** (kode Python, ada tanda `[ ]` di kiri) akan menjalankan kode dan menampilkan
     output (tabel/grafik) tepat di bawah sel tersebut.
4. **Baca setiap output yang muncul** sebelum lanjut ke sel berikutnya — ini bagian penting dari
   belajarnya, bukan cuma menjalankan tanpa mengerti.
5. Kalau ingin menjalankan semua sel sekaligus dari awal: klik menu **"Run All"** di toolbar atas
   notebook (ikon double-play ▶▶).

**Estimasi waktu jalan:** sekitar 1-3 menit untuk seluruh notebook (karena dataset cukup besar,
~540 ribu baris transaksi mentah).

---

## ✅ STEP 7 — Cek Hasil

Setelah notebook selesai dijalankan semua, buka folder `outputs/` di Explorer VSCode — seharusnya
sudah otomatis terisi:
- `transactions_cleaned.csv` — data transaksi setelah cleaning
- `rfm_table.csv` — tabel RFM per pelanggan (sebelum cluster)
- `rfm_clustered.csv` — tabel RFM + label cluster
- `cluster_profile.csv` — ringkasan profil tiap cluster

Grafik (elbow, scatter PCA, scatter 3D, boxplot) muncul langsung di bawah sel notebook yang
bersangkutan — kamu bisa klik kanan pada gambar di notebook untuk **"Save Image As..."** kalau
ingin disimpan sebagai file `.png` terpisah (untuk ditempel di laporan/presentasi).

---


## 🧩 Troubleshooting Cepat

| Masalah | Solusi |
|---|---|
| `ModuleNotFoundError: No module named 'pandas'` (atau library lain) | Kernel yang dipilih bukan dari `venv`. Ulangi Step 5. |
| Notebook tidak menemukan file CSV (`FileNotFoundError`) | Pastikan kamu menjalankan notebook dari lokasi aslinya di folder `notebook/` (jangan pindahkan file `.ipynb` ke folder lain), karena path di kode memakai path relatif `../data/...`. |
| `pip install` gagal/lambat sekali | Cek koneksi internet, atau coba lagi beberapa saat (kadang server pip sedang sibuk). |
| Tombol "Select Kernel" tidak muncul | Pastikan extension **Jupyter** sudah terinstall & VSCode sudah di-reload. |
| Grafik 3D tidak muncul / error terkait `mplot3d` | Pastikan instalasi `matplotlib` sukses lewat `requirements.txt` — kode sudah otomatis mengimpor modul yang dibutuhkan. |
