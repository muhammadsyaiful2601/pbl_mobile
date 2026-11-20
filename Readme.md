# PBL UMKM Hybrid — Sistem Pemesanan & Kasir

Monorepo untuk sistem pemesanan & kasir hybrid `[Nama Cafe / UMKM]`.

## Struktur Folder

```text
pbl-umkm-hybrid/
├── mobile_app/                     # Flutter (Kasir, Tukang Masak, Owner)
├── web_qr/                         # Website Pelanggan (Scan QR Meja & Menu Ticket)
├── backend_functions/              # Cloud Functions / Webhook (Payment Gateway, WA API)
├── .gitignore
└── Readme.md
```

## Arsitektur

Hybrid Monorepo:
- `mobile_app/` — berbasis **Flutter** (Kasir POS, Dapur KDS, Owner Report, Auth, Splash/Info)
- `web_qr/` — berbasis **Web** (Katalog Menu, Keranjang, Form WhatsApp, E-Ticket, Ticket Tracker, Info/FAQ)
- `backend_functions/` — Cloud Functions / Webhook (database real-time, QRIS, WhatsApp API)

Integrasi utama: Real-Time Database, Ticket Code System, Payment Gateway (QRIS), WhatsApp API.

## Setup Awal (Tahap 1 — Selesai)

- [x] Repository Git siap (branch `pbl`)
- [x] Struktur folder monorepo dibuat sesuai PRD
- [x] `mobile_app/` — Flutter project terscaffold (`flutter create`)
- [x] `web_qr/` — project web dasar (package.json + vite)
- [x] `backend_functions/` — folder siap untuk Cloud Function / Webhook
- [x] `.gitignore` dikonfigurasi (Flutter + Node/Web)
- [x] Dependensi terpasang (`flutter pub get` + `npm install`)

Belum dilakukan (tahap berikutnya, core sistem):
- Konfigurasi database real-time (Firebase/Supabase) + security rules
- Model data (`OrderModel`, `MenuModel`, `UserModel`)
- Generator Ticket Code otomatis
- Integrasi Payment Gateway (QRIS) & webhook
- WhatsApp API Gateway

## Pembagian Tugas (per PRD)

| Anggota | Role | Folder yang diizinkan |
|---------|------|----------------------|
| Muhammad Syaiful | Ketua & Lead Full-Stack/Backend | `mobile_app/lib/core/`, `models/`, `services/`, `web_qr/src/config/`, `backend_functions/` |
| Tika | Frontend Web QR | `web_qr/src/pages/order_menu/` |
| Arum | Frontend Web QR | `web_qr/src/pages/ticket_tracker/` |
| Rahma | Flutter Mobile | `mobile_app/lib/features/auth/`, `kasir_pos/` |
| Hafiz | Flutter Mobile | `mobile_app/lib/features/dapur_kds/`, `owner_report/` |
| Widia | UI/UX & Info | `mobile_app/lib/features/splash_info/`, `web_qr/src/pages/info_faq/`, `web_qr/src/assets/` |

## Cara Jalankan

```bash
# Mobile (Flutter)
cd mobile_app
flutter pub get
flutter run

# Web QR
cd web_qr
npm install
npm run dev
```
