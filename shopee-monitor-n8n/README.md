# 🛒 Shopee Monitor n8n

Sistem monitoring toko Shopee otomatis menggunakan [n8n](https://n8n.io) — platform workflow automation open-source.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![n8n](https://img.shields.io/badge/n8n-workflow-orange)](https://n8n.io)
[![Shopee API](https://img.shields.io/badge/Shopee-Open%20Platform-red)](https://open.shopee.com)

---

## ✨ Fitur

| Workflow | Deskripsi | Jadwal |
|----------|-----------|--------|
| 🛒 Monitor Pesanan Baru | Notifikasi saat ada pesanan masuk (status: READY_TO_SHIP) | Setiap 5 menit |
| 📊 Laporan Penjualan | Rekap harian & mingguan pesanan selesai + pendapatan | Harian 08:00 & Senin 08:00 WIB |
| ⭐ Monitor Ulasan Produk | Deteksi ulasan baru dari pembeli | Setiap 30 menit |
| ⚠️ Alert Stok Hampir Habis | Peringatan ketika stok produk di bawah ambang batas | Setiap 1 jam |

**Channel Notifikasi:** Telegram Bot, Email (SMTP), WhatsApp (via Fonnte)

---

## 📋 Prasyarat

- [Docker](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/install/)
- Akun [Shopee Partner](https://open.shopee.com) dengan akses API (untuk mendapatkan `partner_id` dan `partner_key`)
- Telegram Bot Token (dari [@BotFather](https://t.me/BotFather)) — opsional
- Akun Gmail atau SMTP server — opsional
- Token [Fonnte](https://fonnte.com) untuk WhatsApp — opsional

---

## 🔑 Setup Shopee API

1. Daftar di [Shopee Open Platform](https://open.shopee.com) sebagai partner
2. Buat aplikasi baru di dashboard partner — catat `Partner ID` dan `Partner Key`
3. Jalankan OAuth untuk mendapatkan `access_token` dan `shop_id`:
   - Arahkan seller ke URL otorisasi Shopee (lihat dokumentasi partner)
   - Tukarkan authorization code dengan `access_token` menggunakan endpoint `/api/v2/auth/token/get`
4. Catat `access_token`, `refresh_token`, dan `shop_id`

> **Catatan:** `access_token` Shopee berlaku selama 30 hari. Perbarui secara manual menggunakan `refresh_token` via endpoint `/api/v2/auth/access_token/get` sebelum kedaluwarsa.

---

## 🚀 Instalasi

```bash
# 1. Clone repositori
git clone https://github.com/akuipul/shopee-monitor-n8n.git
cd shopee-monitor-n8n

# 2. Salin file konfigurasi dan isi dengan kredensial Anda
cp .env.example .env
nano .env

# 3. Jalankan n8n dan PostgreSQL
docker compose up -d

# 4. Buka n8n di browser
# http://localhost:5678
```

---

## 📥 Import Workflow ke n8n

1. Buka n8n di `http://localhost:5678`
2. Login dengan username & password yang ada di `.env`
3. Klik **+** (New Workflow) → **Import from file**
4. Import satu per satu file dari folder `workflows/`:
   - `shopee-new-orders.json`
   - `shopee-sales-report.json`
   - `shopee-reviews-monitor.json`
   - `shopee-stock-alert.json`
5. Setelah import, aktifkan workflow dengan toggle **Active**

---

## 🔐 Setup Credential di n8n

Setelah import workflow, buat credential berikut di **Settings → Credentials**:

### Telegram Bot
- Nama credential: `Shopee Telegram Bot`
- Tipe: `Telegram API`
- Access Token: isi dari nilai `TELEGRAM_BOT_TOKEN` di `.env`

### SMTP (Email)
- Nama credential: `Shopee SMTP`
- Tipe: `SMTP`
- Host, Port, User, Password: sesuaikan dengan nilai di `.env`

> WhatsApp menggunakan HTTP Request langsung dengan token dari env — tidak perlu credential tambahan.

---

## ⚙️ Konfigurasi

Edit file `.env` untuk mengatur semua parameter:

```env
# Shopee API — wajib diisi
SHOPEE_PARTNER_ID=...
SHOPEE_PARTNER_KEY=...
SHOPEE_SHOP_ID=...
SHOPEE_ACCESS_TOKEN=...

# Ambang batas stok (default: 10 unit)
STOCK_ALERT_THRESHOLD=10
```

---

## 🏗️ Arsitektur

```
┌─────────────────────────────────────────────┐
│                  n8n Container               │
│                                             │
│  ┌──────────────┐    ┌───────────────────┐  │
│  │ Schedule     │───▶│ Shopee API        │  │
│  │ Trigger      │    │ (HMAC-SHA256 Sign)│  │
│  └──────────────┘    └────────┬──────────┘  │
│                               │              │
│                    ┌──────────▼──────────┐  │
│                    │  Logic & Formatting  │  │
│                    └──────────┬──────────┘  │
│                               │              │
│           ┌───────────────────┼──────────┐  │
│           ▼                   ▼          ▼  │
│      [Telegram]           [Email]   [WhatsApp]│
└─────────────────────────────────────────────┘
                    │
          ┌─────────▼──────────┐
          │  PostgreSQL DB     │
          │  (n8n state store) │
          └────────────────────┘
```

---

## 🔧 Troubleshooting

| Masalah | Solusi |
|---------|--------|
| `invalid signature` | Pastikan `SHOPEE_PARTNER_KEY` dan `SHOPEE_SHOP_ID` sudah benar |
| `access token expired` | Perbarui `SHOPEE_ACCESS_TOKEN` di `.env` lalu restart container |
| Telegram tidak terkirim | Cek `TELEGRAM_BOT_TOKEN` dan `TELEGRAM_CHAT_ID` |
| Email gagal terkirim | Untuk Gmail, gunakan App Password bukan password biasa |
| n8n tidak bisa akses DB | Tunggu PostgreSQL siap (healthcheck), coba `docker compose restart n8n` |

---

## 📄 Lisensi

MIT License

---

## English Summary

This repository contains n8n workflow templates for automated Shopee store monitoring:
- **New Orders**: polls every 5 minutes for READY_TO_SHIP orders
- **Sales Report**: daily & weekly aggregated revenue report at 08:00 WIB
- **Reviews Monitor**: detects new product reviews every 30 minutes using static workflow data
- **Stock Alert**: checks inventory levels every hour and alerts when below threshold

All workflows use Shopee Open Platform API v2 with HMAC-SHA256 request signing.
Notifications are sent via Telegram, Email (SMTP), and WhatsApp (Fonnte).
Infrastructure: n8n + PostgreSQL via Docker Compose, timezone set to Asia/Jakarta.
