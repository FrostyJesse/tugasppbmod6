# PPB-Modul6 – LiveTemp IoT Temperature Monitoring

> **Kelompok 10** · Praktikum Pemrograman Perangkat Bergerak (PPB) · Teknik Komputer · Universitas Diponegoro 2026

---

## Anggota Kelompok

| No | Nama | NIM |
|----|------|-----|
| 1 | Jesse Timothy Halomoan Damanik | 21120124120041 |
| 2 | Yassar Fawwas Hernando | 21120124120024 |
| 3 | Mohamad Rifqi Adi Prayoga | 21120124120004 |
| 4 | Danish Alfarisy | 21120124120006 |

---

## Deskripsi

**LiveTemp** adalah aplikasi mobile berbasis **Expo / React Native** untuk monitoring suhu IoT secara real-time. Aplikasi ini:

- Menerima data suhu dari sensor melalui protokol **MQTT (HiveMQ)**
- Menyimpan data alarm (suhu melewati threshold) ke **Neon PostgreSQL**
- Menampilkan riwayat data dengan **pagination**
- Dilengkapi dengan **Custom Splash Screen**, **Halaman Profile**, dan **animasi UI Sphere** yang futuristik

---

## Teknologi

| Layer | Teknologi |
|-------|-----------|
| Mobile Frontend | Expo SDK 58 · React Native 0.88 |
| Navigation | React Navigation Bottom Tabs v7 |
| Animasi | React Native Reanimated 4.7.1 · React Native Skia 2.15.1 |
| Protokol IoT | MQTT via WebSocket (HiveMQ public broker) |
| Backend | Node.js · Express 5 |
| Database | Neon Serverless PostgreSQL |
| Sensor Simulator | Node.js · mqtt library |

---

## Struktur Proyek

```
PPB-Modul6/
├── app/                    ← Expo React Native app
│   ├── src/
│   │   ├── components/
│   │   │   ├── DataTable.js
│   │   │   ├── SplashScreen.js   ← Tugas 7.6.1
│   │   │   └── TempOrb.js        ← Tugas 7.6.4 (modifikasi sphere)
│   │   ├── hooks/
│   │   │   └── useMqttSensor.js
│   │   ├── screens/
│   │   │   ├── ControlScreen.js
│   │   │   ├── MonitoringScreen.js  ← Tugas 7.6.2 (pagination)
│   │   │   └── ProfileScreen.js    ← Tugas 7.6.3
│   │   ├── services/
│   │   │   ├── api.js
│   │   │   └── config.js
│   │   └── theme/
│   │       └── tokens.js
│   ├── App.js              ← Entry point + Splash Screen + Profile Tab
│   └── app.json            ← Konfigurasi Expo + MQTT + Backend URL
├── backend/                ← Express.js REST API
│   ├── src/
│   │   ├── config/database.js
│   │   ├── controllers/
│   │   ├── models/         ← Pagination di readingsModel.js
│   │   └── routes/
│   ├── .env                ← DATABASE_URL + PORT (tidak di-push ke GitHub)
│   └── .env.example
├── sensor-simulator/       ← MQTT publisher Node.js
│   └── index.js
├── database/
│   └── schema.sql          ← DDL tabel sensor_readings + threshold_settings
├── documentation/
│   ├── screenshots/        ← Screenshot hasil pengujian
│   └── report/
└── firewall/windows/       ← Script aturan firewall port 5002
```

---

## Cara Menjalankan Aplikasi

### Prasyarat

- Node.js 20+
- Expo Go di smartphone (tersambung ke LAN yang sama)
- Akun Neon PostgreSQL (buat di https://neon.tech)

### 1. Clone Repository

```bash
git clone https://github.com/gavrilasa/PPB-Modul6.git
cd PPB-Modul6
```

### 2. Konfigurasi Database

Buat file `backend/.env` berdasarkan `backend/.env.example`:

```env
DATABASE_URL=postgresql://USER:PASSWORD@HOST/neondb?sslmode=require
PORT=5002
```

Jalankan DDL schema di SQL Editor Neon:

```bash
# Isi dari database/schema.sql
```

### 3. Jalankan Backend (Terminal 1)

```bash
cd backend
npm install
npm run dev
```

Backend akan berjalan di `http://0.0.0.0:5002`

### 4. Jalankan Sensor Simulator (Terminal 2)

```bash
cd sensor-simulator
npm install
npm start
```

Simulator akan mempublikasikan data suhu setiap 5 detik ke topik MQTT `ppb/kel10/iot/temperature`.

### 5. Jalankan Expo App (Terminal 3)

```bash
cd app
npm install
npm start
```

Scan QR Code menggunakan **Expo Go** di smartphone. Pastikan laptop dan smartphone berada di jaringan LAN yang sama.

---

## Konfigurasi Backend

| Parameter | Nilai |
|-----------|-------|
| Backend URL | `http://192.168.1.17:5002` |
| Port | `5002` |

> Ganti `192.168.1.17` dengan IP LAN aktif laptop Anda saat menjalankan aplikasi.

---

## Konfigurasi Database (Neon PostgreSQL)

Dua tabel wajib tersedia di schema `public`:

- **`sensor_readings`** – menyimpan data suhu triggered alarm (temperature ≥ threshold)
- **`threshold_settings`** – menyimpan riwayat batas ambang suhu

Schema lengkap: [`database/schema.sql`](database/schema.sql)

---

## Konfigurasi MQTT

| Parameter | Nilai |
|-----------|-------|
| Broker | `broker.hivemq.com` |
| WebSocket URL (Frontend) | `wss://broker.hivemq.com:8884/mqtt` |
| TCP URL (Simulator) | `mqtt://broker.hivemq.com:1883` |
| Topic | `ppb/kel10/iot/temperature` |

---

## Fitur Tambahan Tugas 7.1

### 7.6.1 – Custom Splash Screen

Splash screen modern menampilkan:
- Logo aplikasi (ikon termometer) dengan animasi ring pulsing
- Nama aplikasi **LiveTemp**
- Badge identitas Kelompok 10
- Animasi loading titik-titik
- Transisi fade keluar sebelum masuk ke halaman utama

**File:** `app/src/components/SplashScreen.js`

### 7.6.2 – Pagination Monitoring

Halaman Monitoring menampilkan data triggered alarm dengan pagination:
- 5 data per halaman
- Tombol **Sebelumnya** (nonaktif di halaman pertama)
- Tombol **Berikutnya** (nonaktif di halaman terakhir)
- Info halaman aktif dan total data
- Backend mendukung query `?page=N&limit=5`

**File:** `app/src/screens/MonitoringScreen.js`, `backend/src/models/readingsModel.js`

### 7.6.3 – Halaman Profile

Halaman baru yang dapat diakses via tab bawah, menampilkan:
- Identitas Kelompok 10 beserta nomor kelompok
- Kartu anggota dengan nama lengkap, NIM, dan avatar inisial
- Informasi sistem (framework, database, protokol, topik MQTT)

**File:** `app/src/screens/ProfileScreen.js`

### 7.6.4 – UI Animation Sphere

Modifikasi `TempOrb.js` dengan:
- Partikel foton 3-layer (outer aura → energy corona → solid core)
- Gelombang plasma permukaan organis (`sin wave` sepanjang sumbu Y)
- Wobble giroskopik dinamis pada sumbu tilt
- Fade aperture halus di zona tampilan suhu
- Ring ambient telemetry ganda di latar sphere
- Kecepatan rotasi sedikit ditingkatkan (0.18 rad/s)

**File:** `app/src/components/TempOrb.js`

---

## Endpoint Backend

| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| GET | `/api/readings` | Ambil 100 data terbaru (tanpa pagination) |
| GET | `/api/readings?page=N&limit=M` | Ambil data dengan pagination |
| POST | `/api/readings` | Simpan data suhu baru |
| GET | `/api/readings/latest` | Data suhu terbaru |
| GET | `/api/thresholds` | Daftar riwayat threshold |
| POST | `/api/thresholds` | Simpan threshold baru |
| GET | `/api/thresholds/latest` | Threshold terbaru |

---

## Keamanan

- File `backend/.env` **tidak** di-push ke GitHub (tercantum dalam `.gitignore`)
- Credential database **tidak** dikirim ke frontend
- Frontend hanya berkomunikasi dengan backend melalui REST API

---

## Firewall (Windows)

Untuk memastikan Expo Go di smartphone dapat mengakses backend:

```bat
# Jalankan sebagai Administrator
firewall\windows\add_firewall_rule.bat
```

---

## Lisensi

Proyek ini dibuat untuk keperluan praktikum akademik.
