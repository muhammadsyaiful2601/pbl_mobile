# Product Requirements Document (PRD) & Pembagian Tugas Tim PBL
## [DRAFT MENTAH] Sistem Pemesanan & Kasir Hybrid `[Nama Cafe / UMKM]`
*(Catatan: Dokumen ini masih berstatus draft awal/mentah karena nama resmi cafe/UMKM mitra belum ditentukan. Silakan ganti placeholder `[Nama Cafe / UMKM]` saat nama usaha sudah final).*

| Atribut Dokumen | Keterangan |
| :--- | :--- |
| **Status Dokumen** | Draft / Mentah (Baseline Pengembangan) |
| **Nama Usaha (Sementara)** | `[Nama Cafe / UMKM]` |
| **Ketua & Lead Developer** | Muhammad Syaiful |
| **Anggota Tim** | Tika, Arum, Rahma, Hafiz, Widia (Formasi Lengkap 6 Orang) |
| **Arsitektur Sistem** | Hybrid Monorepo (`mobile_app/` berbasis Flutter + `web_qr/` berbasis Web) |
| **Aktor Sistem** | Pembeli, Kasir, Tukang Masak (Dapur), Owner |
| **Integrasi Utama** | Real-Time Database, Ticket Code System, Payment Gateway (QRIS), WhatsApp API |

---

## 1. Gambaran Umum & Alur Operasional Sistem

### 1.1 Latar Belakang & Tujuan
`[Nama Cafe / UMKM]` membutuhkan sistem pemesanan dan kasir terintegrasi untuk mengatasi antrean saat jam ramai tanpa menghilangkan kebiasaan pelanggan yang ingin memesan langsung atau membayar tunai (*cash*).

### 1.2 Alur Sistem End-to-End
1. **Dua Jalur Pemesanan Pelanggan:**
   * **Opsi A — Pesan Langsung di Meja Kasir (Walk-In):** Pelanggan datang ke kasir, memesan menu secara langsung, dan membayar tunai (*cash*) atau QRIS melalui aplikasi Flutter Kasir.
   * **Opsi B — Scan QR Code di Meja (Web QR):** Pelanggan memindai QR Code di meja menggunakan kamera HP dan langsung diarahkan ke halaman **Website Pemesanan (`web_qr`)** tanpa perlu mengunduh aplikasi.
2. **Transaksi & Pengisian WhatsApp di Website:**
   * Di website, pelanggan memilih menu, mengisi **Nama Pemesan**, mengisi **Nomor WhatsApp aktif**, serta memilih metode pembayaran (**QRIS / Payment Gateway** atau **Tunai di Kasir**).
   * Website terhubung secara *real-time* dengan aplikasi di meja kasir.
3. **Penerbitan Ticket Code & Pemantauan via Menu Ticket:**
   * Setelah pesanan dibuat, sistem menerbitkan **Ticket Code unik** (contoh: `TKT-101`).
   * Jika membayar via **QRIS**, tiket otomatis berstatus `Lunas` dan langsung diteruskan ke layar **Tukang Masak (Dapur)**. Jika memilih **Tunai**, pembeli menunjukkan *Ticket Code* ke kasir untuk dibayar terlebih dahulu sebelum diteruskan ke dapur.
   * Pembeli memasukkan *Ticket Code* miliknya ke dalam **Menu Ticket** di website untuk memantau progres pesanan secara *real-time* (`Dibuat` $\rightarrow$ `Lunas` $\rightarrow$ `Dimasak` $\rightarrow$ `Siap Diambil`).
4. **Notifikasi WhatsApp Otomatis Saat Makanan Siap:**
   * Ketika Tukang Masak menekan tombol **Pesanan Siap Diambil**, sistem otomatis mengirimkan pesan **WhatsApp** ke nomor pembeli yang diinput saat pemesanan bahwa makanan sudah matang dan siap diambil.
5. **Pelaporan Owner:**
   * Seluruh transaksi (baik dari Web QR maupun Kasir Manual) otomatis terekap dalam dashboard laporan milik **Owner**.

---

## 2. Struktur Folder Proyek (Setup Awal oleh Muhammad Syaiful)

Proyek menggunakan struktur *monorepo* agar memisahkan kode aplikasi mobile staf dengan website pelanggan, namun tetap terhubung ke satu *backend/database* yang sama. **Tahap inisialisasi folder ini dikerjakan penuh oleh Muhammad Syaiful di awal proyek.**

```text
pbl-umkm-hybrid/
├── mobile_app/                     # Khusus Aplikasi Flutter (Kasir, Tukang Masak, Owner)
│   └── lib/
│       ├── main.dart
│       ├── core/                   # [Syaiful] Config Firebase/Supabase, Theme, Ticket Generator
│       ├── models/                 # [Syaiful] OrderModel, MenuModel, UserModel
│       ├── services/               # [Syaiful] FirestoreService, PaymentService, WhatsAppService
│       └── features/
│           ├── splash_info/        # [Widia] Splash Screen & Halaman Profil Cafe
│           ├── auth/               # [Rahma] Login & Role Redirect (Kasir, Masak, Owner)
│           ├── kasir_pos/          # [Rahma] POS Manual, Kalkulator Kembalian, Validasi Tiket
│           ├── dapur_kds/          # [Hafiz] Layar Tukang Masak & Update Status Masak
│           └── owner_report/       # [Hafiz] Dashboard KPI Owner, Grafik & Export Laporan
│
├── web_qr/                         # Khusus Website Pelanggan (Scan QR Meja & Menu Ticket)
│   ├── src/
│   │   ├── config/                 # [Syaiful] Koneksi Database Real-Time & API Endpoint
│   │   ├── pages/
│   │   │   ├── order_menu/         # [Tika] Scan Parameter Meja, Katalog Menu, Keranjang & Form WA
│   │   │   ├── ticket_tracker/     # [Arum] Kartu E-Ticket Digital & Menu Lacak Ticket Code
│   │   │   └── info_faq/           # [Widia] Halaman Panduan Cara Pesan & Profil Cafe
│   │   └── assets/                 # [Widia] Gambar Menu, Logo Sementara & Ikon UI
│
└── backend_functions/              # [Syaiful] Cloud Functions / Webhook Payment Gateway & WA API
```

---

## 3. Spesifikasi Data Utama (`orders`)

```json
{
  "ticketCode": "TKT-101",
  "source": "Web QR",
  "tableNumber": "Meja 04",
  "customerName": "Budi",
  "whatsappNumber": "6281234567890",
  "items": [
    { "menuId": "m1", "name": "Kopi Susu Gula Aren", "price": 18000, "qty": 2 }
  ],
  "note": "Es sedikit",
  "paymentMethod": "QRIS",
  "paymentStatus": "PAID",
  "orderStatus": "siap_diambil",
  "waNotificationSent": true,
  "totalAmount": 36000,
  "createdAt": "2026-11-19T18:30:00+07:00"
}
```

---

## 4. Pembagian Tugas Anggota (Coding + Laporan + Batasan AI Agent)

> **PENTING UNTUK PENGGUNAAN AI AGENT (Cursor / Windsurf / Copilot / ChatGPT / Gemini):**
> Setiap anggota tim wajib memberikan instruksi batasan folder di bawah ini kepada AI Agent masing-masing agar AI **hanya** membuat/mengedit file di bagiannya sendiri dan **tidak mengambil alih atau menimpa** kode milik anggota lain.

---

### 1. Muhammad Syaiful — Ketua Tim & Lead Full-Stack / Backend Developer
* **Tugas Setup Awal & Coding:**
  1. Membuat repository Git dan menyiapkan kerangka awal proyek untuk folder **`mobile_app/` (Flutter)** dan **`web_qr/` (Website)**.
  2. Mengonfigurasi koneksi database *real-time* (Firebase Firestore / Supabase), *security rules*, serta model data (`OrderModel`, `MenuModel`).
  3. Membuat fungsi *core* generator **Ticket Code** otomatis (`TKT-101`, dst.).
  4. Mengintegrasikan **Payment Gateway (QRIS Sandbox)** dan *webhook* verifikasi pembayaran otomatis.
  5. Membangun *Cloud Function / Webhook* **WhatsApp API Gateway** untuk mengirim pesan otomatis ke nomor WhatsApp pembeli saat status tiket berubah menjadi `siap_diambil`.
* **Tugas Laporan PBL:**
  * **Bab 1 (Pendahuluan)**, **Bab 3 (Arsitektur Sistem Hybrid & Skema Database)**, serta *review* & penggabungan akhir seluruh laporan.
* **Instruksi Batasan AI Agent (Untuk Muhammad Syaiful):**
  * **BOLEH dikerjakan AI:** File di dalam `mobile_app/lib/core/`, `mobile_app/lib/models/`, `mobile_app/lib/services/`, `web_qr/src/config/`, dan `backend_functions/`.
  * **DILARANG dikerjakan AI:** Jangan membuatkan UI halaman fitur milik Tika (`order_menu`), Arum (`ticket_tracker`), Rahma (`auth`, `kasir_pos`), Hafiz (`dapur_kds`, `owner_report`), maupun Widia (`splash_info`, `info_faq`).

---

### 2. Tika — Frontend Developer Web QR (Katalog, Keranjang & Form WhatsApp)
* **Tugas Coding:**
  1. Membangun halaman pemesanan pada **`web_qr/`** yang membaca parameter nomor meja otomatis dari URL QR (contoh: `?meja=04`).
  2. Menampilkan katalog menu, pencarian, dan filter kategori (*Makanan, Minuman, Camilan*).
  3. Membangun fitur keranjang belanja dan form checkout yang mewajibkan input **Nama Pemesan** dan validasi **Nomor WhatsApp aktif** (`08xx` / `628xx`), serta pilihan metode bayar (**QRIS** atau **Tunai di Kasir**).
  4. Menyimpan data pesanan baru ke database dan mengarahkan pembeli ke halaman E-Ticket milik Arum setelah checkout berhasil.
* **Tugas Laporan PBL:**
  * **Bab 2 (Tinjauan Pustaka)** dan **Bab 4 (Implementasi – Bagian Modul Pemesanan Web QR)**.
* **Instruksi Batasan AI Agent (Untuk Tika):**
  * **BOLEH dikerjakan AI:** Hanya file di dalam folder `web_qr/src/pages/order_menu/`.
  * **DILARANG dikerjakan AI:** Jangan mengubah konfigurasi database di `web_qr/src/config/`, jangan membuat halaman pelacakan tiket (`ticket_tracker`), dan jangan menyentuh folder `mobile_app/`.

---

### 3. Arum — Frontend Developer Web QR (E-Ticket & Menu Lacak Ticket Code)
* **Tugas Coding:**
  1. Membangun tampilan kartu **E-Ticket Digital** setelah pembeli selesai memesan (menampilkan *Ticket Code* besar, barcode/QR tiket, tombol salin kode, dan instruksi bayar jika memilih tunai).
  2. Membangun halaman **Menu Ticket** pada **`web_qr/`** tempat pembeli memasukkan/mengetikkan *Ticket Code* mereka (contoh: `TKT-101`).
  3. Menampilkan status pesanan secara *real-time* menggunakan *stepper* 4 tahap (`Dibuat` $\rightarrow$ `Lunas` $\rightarrow$ `Dimasak` $\rightarrow$ `Siap Diambil`) serta tombol konfirmasi pesanan diterima.
* **Tugas Laporan PBL:**
  * **Bab 4 (Implementasi – Bagian Modul E-Ticket & Pelacakan Ticket Code)** serta dokumentasi alur *real-time* pembeli.
* **Instruksi Batasan AI Agent (Untuk Arum):**
  * **BOLEH dikerjakan AI:** Hanya file di dalam folder `web_qr/src/pages/ticket_tracker/`.
  * **DILARANG dikerjakan AI:** Jangan mengubah halaman katalog/checkout (`order_menu`), jangan mengubah `web_qr/src/config/`, dan jangan menyentuh folder `mobile_app/`.

---

### 4. Rahma — Flutter Mobile Developer (Autentikasi Staf & Modul Kasir POS)
* **Tugas Coding:**
  1. Membangun halaman **Login Staf** di **`mobile_app/`** dan pengalihan halaman otomatis berdasarkan peran (`Kasir`, `Tukang Masak`, `Owner`).
  2. Membangun layar **POS Kasir Manual** untuk pelanggan yang memesan langsung di meja kasir (pilih menu, input nama/WA opsional, keranjang kasir).
  3. Membuat fitur **Kalkulator Kembalian Tunai** (input uang diterima, tombol pecahan cepat, hitung kembalian otomatis) dan modal struk *Ticket Code*.
  4. Membangun tab **Validasi Tiket Tunai** untuk mencari *Ticket Code* dari pesanan Web QR yang memilih bayar tunai di kasir dan mengubah statusnya menjadi lunas agar masuk ke dapur.
* **Tugas Laporan PBL:**
  * **Bab 3 (Diagram UML: Use Case Diagram & Activity Diagram)** dan **Bab 4 (Implementasi – Bagian Modul Login & Kasir POS)**.
* **Instruksi Batasan AI Agent (Untuk Rahma):**
  * **BOLEH dikerjakan AI:** Hanya file di dalam folder `mobile_app/lib/features/auth/` dan `mobile_app/lib/features/kasir_pos/`.
  * **DILARANG dikerjakan AI:** Jangan mengubah `core/`, `models/`, `services/`, layar dapur (`dapur_kds`), layar owner (`owner_report`), maupun folder `web_qr/`.

---

### 5. Hafiz — Flutter Mobile Developer (Layar Dapur KDS & Dashboard Owner)
* **Tugas Coding:**
  1. Membangun layar **Tukang Masak / Kitchen Display System (KDS)** di **`mobile_app/`** yang hanya menampilkan tiket berstatus lunas (`PAID`) dengan *Ticket Code* berukuran besar dan catatan dapur.
  2. Membuat tombol kontrol status masak (`Mulai Masak` $\rightarrow$ `Selesai & Siap Diambil`) yang memicu pengiriman notifikasi WhatsApp otomatis ke pembeli.
  3. Membangun **Dashboard Laporan Owner** yang menampilkan KPI omzet *real-time*, perbandingan pemasukan Tunai vs QRIS, grafik menu terlaris, log transaksi lengkap dengan *Ticket Code*, serta fitur *export* laporan (CSV/PDF).
* **Tugas Laporan PBL:**
  * **Bab 4 (Implementasi – Bagian Modul Dapur KDS & Dashboard Owner)** serta **Bab 5 (Pengujian Sistem / Tabel Black-Box Testing)**.
* **Instruksi Batasan AI Agent (Untuk Hafiz):**
  * **BOLEH dikerjakan AI:** Hanya file di dalam folder `mobile_app/lib/features/dapur_kds/` dan `mobile_app/lib/features/owner_report/`.
  * **DILARANG dikerjakan AI:** Jangan mengubah `core/`, `models/`, `services/`, modul kasir (`kasir_pos`), maupun folder `web_qr/`.

---

### 6. Widia — UI/UX Support, Modul Informasi & Dokumentasi Panduan
* **Tugas Coding:**
  1. Menyiapkan aset gambar menu, ikon kategori, dan konstanta tema warna/tipografi di **`web_qr/src/assets/`**.
  2. Membangun halaman pembuka (**Splash Screen**) serta halaman informasi **Profil `[Nama Cafe / UMKM]`** di **`mobile_app/lib/features/splash_info/`**.
  3. Membangun halaman statis **Panduan Cara Pesan & FAQ** di **`web_qr/src/pages/info_faq/`** untuk membantu pelanggan memahami cara scan QR dan penggunaan *Menu Ticket*.
* **Tugas Laporan PBL:**
  * **Bab 5 (Penutup: Kesimpulan & Saran)**, **Buku Panduan Pengguna (*User Manual*)** untuk 4 aktor, dan **Lampiran** (daftar pustaka & desain *tent-card* QR Code meja).
* **Instruksi Batasan AI Agent (Untuk Widia):**
  * **BOLEH dikerjakan AI:** Hanya file di dalam `mobile_app/lib/features/splash_info/`, `web_qr/src/pages/info_faq/`, dan `web_qr/src/assets/`.
  * **DILARANG dikerjakan AI:** Jangan menyentuh logika transaksi, database, kasir, dapur, maupun pelacakan tiket.