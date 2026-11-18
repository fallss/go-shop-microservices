# Panduan Lengkap untuk Pemula: Memahami Proyek Shop Microservices

Selamat datang! Dokumen ini dibuat khusus untuk Anda yang baru pertama kali belajar backend dan microservices, dengan bahasa yang sederhana, analogi kehidupan sehari-hari, dan penjelasan langkah demi langkah.

---

## 1. Apa Sebenarnya Proyek Ini?

Proyek ini adalah **simulasi sistem belanja online (e-commerce)** mini yang dibuat menggunakan bahasa pemrograman **Go (Golang)**.

Bayangkan Anda sedang berbelanja di minimarket atau marketplace:
1. Ada katalog barang yang dijual (nama barang, harga, dan sisa stok).
2. Ada kasir tempat pembeli memesan barang, menghitung total bayar, dan membatalkan pesanan jika berubah pikiran.

Proyek ini membuat sistem backend (mesin di balik layar) untuk menangani dua tugas di atas.

---

## 2. Analogi Dunia Nyata: Mengapa Dibagi 2 Service?

Di proyek ini, sistem tidak dibuat dalam satu program besar (*Monolith*), melainkan dipecah menjadi dua program kecil mandiri yang disebut **Microservices**:

| Nama Service | Analogi di Dunia Nyata | Tugas & Tanggung Jawab | Berjalan di Port |
| :--- | :--- | :--- | :--- |
| **`product-service`** | **Petugas Gudang / Bagian Stok** | Menyimpan daftar barang, harga barang, dan sisa stok barang di rak. | `:8081` |
| **`order-service`** | **Kasir / Meja Pemesanan** | Menerima pesanan pembeli, mengecek stok ke petugas gudang, membuat nota pesanan, dan melayani pembatalan order. | `:8082` |

### Bagaimana Keduanya Berkomunikasi?

```text
 Pembeli / User
      │
      ▼  (1. "Saya mau beli Kaos 1 pcs")
┌─────────────────────────────────┐
│          ORDER-SERVICE          │ (Kasir - Port 8082)
└─────────────────────────────────┘
      │
      ▼  (2. Tanya ke gudang: "Kaos masih ada stok? Harganya berapa?")
┌─────────────────────────────────┐
│         PRODUCT-SERVICE         │ (Gudang - Port 8081)
└─────────────────────────────────┘
      │
      ▼  (3. Jawab: "Ada! Stok sisa 50, harga Rp 79.000")
┌─────────────────────────────────┐
│          ORDER-SERVICE          │
└─────────────────────────────────┘
      │
      ▼  (4. "Pesanan nomor ord-1 berhasil dibuat!")
 Pembeli / User
```

---

## 3. Kamus Istilah untuk Pemula

Bagi yang baru pertama kali melihat istilah teknis ini, berikut artinya:

### A. HTTP Method
- **`GET`**: Digunakan saat Anda ingin **mengambil/melihat** data (seperti melihat katalog baju atau melihat isi nota). Tidak mengubah isi data.
- **`POST`**: Digunakan saat Anda ingin **membuat atau mengubah data baru** (seperti mengirim formulir pemesanan baru atau membatalkan pesanan).

### B. HTTP Status Code (Kode Respon Server)
Saat Anda mengirim perintah, server akan membalas dengan kode angka 3 digit:
- **`200 OK`**: Berhasil! Permintaan Anda diterima dan data berhasil diambil.
- **`201 Created`**: Berhasil dibuat! Data pesanan baru berhasil dicatat di sistem.
- **`400 Bad Request`**: Permintaan salah format (misal: JSON rusak atau jumlah barang 0).
- **`404 Not Found`**: Data tidak ditemukan (misal: mencari order `ord-999` yang memang tidak pernah ada).
- **`409 Conflict`**: Terjadi tabrakan aturan (misal: membatalkan order yang statusnya sudah pernah dibatalkan, atau stok barang habis).
- **`502 Bad Gateway`**: **Penyebab error yang tadi Anda alami!** Artinya: *order-service* (kasir) mencoba menelepon *product-service* (gudang), tetapi teleponnya tidak diangkat karena *product-service* belum dinyalakan.

---

## 4. Mengapa Tadi Sempat Muncul `502 Bad Gateway`?

Saat Anda pertama kali mencoba tadi, Anda baru menjalankan:
```bash
go run cmd/order-service/main.go  # (Port 8082 menyala)
```
Tetapi Anda **belum** menyalakan:
```bash
go run cmd/product-service/main.go # (Port 8081 masih MATI)
```

Ketika Anda klik **`POST /orders`**, kasir (*order-service*) mencoba menghubungi gudang (*product-service* di `localhost:8081`). Karena gudang sedang tutup/mati, kasir kebingungan dan mengembalikan kode **`502 Bad Gateway (product-service unavailable)`**.

Akibatnya:
- Order **gagal dibuat**.
- Karena gagal dibuat, data order `ord-1` belum tersimpan.
- Sehingga saat Anda mencoba `GET /orders/ord-1` dan `POST /cancel`, server menjawab **`404 Not Found`** karena memang datanya tidak ada.

---

## 5. Panduan Praktik Langkah demi Langkah yang Benar

Ikuti langkah mudah ini untuk melihat seluruh sistem bekerja dengan sempurna:

### Langkah 1: Buka Terminal 1 (Nyalakan Gudang)
Buka terminal di komputer Anda, lalu jalankan:
```bash
cd ~/Downloads/shop-microservices-exp
go run cmd/product-service/main.go
```
*Tunggu sampai muncul tulisan:*
`product-service listening on :8081`

### Langkah 2: Buka Terminal 2 (Nyalakan Kasir)
Buka tab atau jendela terminal baru (jangan matikan terminal 1):
```bash
cd ~/Downloads/shop-microservices-exp
go run cmd/order-service/main.go
```
*Tunggu sampai muncul tulisan:*
`order-service listening on :8082 (product-service: http://localhost:8081)`

> [!TIP]
> **Keduanya harus tetap menyala bersamaan!** Jangan tekan `Ctrl + C` pada kedua terminal tersebut selama Anda ingin melakukan pengujian.

---

### Langkah 3: Menguji Lewat GoLand (`requests.http`)

Sekarang buka aplikasi **GoLand**, buka file **`requests.http`**, lalu klik tombol panah hijau (▶) secara berurutan:

1. **Klik ▶ pada `1. Health Check`**
   - Hasil: `200 OK` ➔ Memastikan kasir hidup dan sehat.
2. **Klik ▶ pada `2. List All Orders`**
   - Hasil: `200 OK []` ➔ Daftar pesanan masih kosong karena belum ada yang memesan.
3. **Klik ▶ pada `3. Create Order`**
   - Hasil: **`201 Created`**! ➔ Kasir mengecek stok `p-1` ke gudang, gudang menjawab stok ada, dan pesanan `ord-1` resmi terbuat!
4. **Klik ▶ pada `4. Get Order by ID`**
   - Hasil: **`200 OK`** ➔ Menampilkan detail order `ord-1` yang tadi baru saja dibuat.
5. **Klik ▶ pada `5. Cancel Order`**
   - Hasil: **`200 OK`** ➔ Status order `ord-1` kini berubah dari `"pending"` menjadi `"cancelled"`.
6. **Klik ▶ pada `6. Cancel Order Lagi`**
   - Hasil: **`409 Conflict`** ➔ Server menolak dengan pesan `"order cannot be cancelled"` karena order tersebut sudah pernah dibatalkan sebelumnya.

---

## 6. Apa Hubungannya dengan Workshop "From Natural Language to Executable Code"?

Workshop yang sedang Anda ikuti mengajarkan paradigma baru dalam dunia programming:
1. **Dulu**: Programmer harus mengetik ratusan baris kode dari nol secara manual.
2. **Sekarang (Era AI Coding Agent)**: Anda sebagai *software engineer* memberikan **instruksi spesifik** dalam bahasa manusia (*Natural Language*), lalu AI Agent menyusun rencana, mengedit kode, dan menjalankan testing.
3. **Peran Manusia Tetap Kunci**: AI bisa saja salah paham jika instruksi ambigu. Seperti di **Slide 2** tadi, AI tidak langsung asal coding pembatalan order, melainkan **mewawancarai Anda terlebih dahulu** untuk memastikan aturan bisnisnya (misal: apa yang terjadi jika order sudah pernah di-cancel? status apa yang dikembalikan?). Setelah jelas, barulah kodenya dieksekusi dan diverifikasi dengan unit test.

---

## 7. Struktur Folder Proyek Ini

Agar Anda tidak bingung saat melihat isi folder di GoLand:
```text
shop-microservices-exp/
├── cmd/
│   ├── product-service/main.go   # Titik awal menjalankan Product Service (port 8081)
│   └── order-service/main.go     # Titik awal menjalankan Order Service (port 8082)
├── internal/
│   ├── httpx/                    # Fungsi pembantu untuk response JSON & Error
│   ├── product/                  # Logika data produk & katalog
│   └── order/                    # Logika pembuatan order, get order, & cancel order
├── docs/                         # Dokumen materi workshop (SPEC, PROMPTS, CHALLENGES)
├── requests.http                 # File dashboard pengujian API untuk GoLand
├── PANDUAN_PEMULA.md             # Dokumen panduan yang sedang Anda baca ini
└── go.mod                        # Konfigurasi dependensi modul Go
```

Selamat mencoba dan bereksplorasi! Jangan ragu bertanya jika ada bagian yang ingin diperjelas.
