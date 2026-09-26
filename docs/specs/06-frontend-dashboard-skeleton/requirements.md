# Spesifikasi Kebutuhan (Requirements) — Fase 6: Frontend: Dashboard Skeleton (React Router)

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk fondasi aplikasi frontend dashboard berbasis React Router, mencakup struktur layout aplikasi (*application shell*), arsitektur routing halaman, layer pengambilan data awal (*data loader* via REST API), serta manajemen koneksi persisten WebSocket client (*live feed provider*).

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-601: Struktur Navigasi & Shell Layout Aplikasi
Sebagai *pengguna*, saya ingin memiliki tata letak dashboard yang responsif dan konsisten dengan sidebar/navbar navigasi agar saya dapat berpindah antara halaman pemantauan langsung (*Live Monitoring*) dan halaman analitik (*Historical Analytics*) dengan mudah.

- **REQ-601-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan tata letak utama (*App Shell*) yang memuat bilah navigasi (Navbar/Sidebar), judul pot aktif, pemilih perangkat (*pot selector*), dan indikator status koneksi server.
- **REQ-601-02 (Event-driven)**: WHEN pengguna mengakses URL root `/`, THE SYSTEM SHALL mengalihkan (*redirect*) pengguna secara otomatis ke halaman `/monitoring`.
- **REQ-601-03 (Event-driven)**: WHEN tautan menu diklik, THE SYSTEM SHALL melakukan pergantian tampilan halaman tanpa memuat ulang browser secara penuh (*client-side routing*).

### US-602: Layer Pengambilan Data REST (React Router Loaders)
Sebagai *klien frontend*, saya ingin mengambil data awal dari backend REST API menggunakan mekanisme *loader* React Router sebelum halaman dirender agar pengguna tidak melihat antarmuka kosong yang tidak stabil (*layout shift*).

- **REQ-602-01 (Event-driven)**: WHEN rute `/monitoring` diakses, THE SYSTEM SHALL memanggil endpoint `GET /api/readings/latest` melalui loader untuk menyediakan data awal metrik sensor.
- **REQ-602-02 (Event-driven)**: WHEN rute `/analytics` diakses, THE SYSTEM SHALL memanggil endpoint `GET /api/analytics/summary` dan `GET /api/readings?range=24h` melalui loader.
- **REQ-602-03 (Unwanted event)**: IF permintaan REST API mengalami kegagalan (koneksi ditolak / status 5xx), THEN THE SYSTEM SHALL merender komponen penanganan error (*Error Boundary*) yang informatif dengan tombol coba lagi (*Retry*).

### US-603: Klien WebSocket & Manajemen Status Koneksi
Sebagai *klien frontend*, saya ingin mempertahankan koneksi persisten ke WebSocket gateway backend dengan pemulihan otomatis jika terputus agar data telemetri real-time dapat terus diterima.

- **REQ-603-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan context atau custom hook (`useWebSocket`) yang menginisialisasi koneksi ke endpoint WebSocket backend (`VITE_WS_URL`).
- **REQ-603-02 (State-driven)**: WHILE status koneksi WebSocket berubah, THE SYSTEM SHALL memperbarui state global koneksi (`connecting`, `connected`, `disconnected`, `reconnecting`).
- **REQ-603-03 (Unwanted event)**: IF koneksi WebSocket terputus, THEN THE SYSTEM SHALL melakukan upaya penyambungan ulang (*auto-reconnect*) secara berkala dengan interval eksponensial (misal 2s, 4s, 8s, maks 30s).

---

## 2. Non-Functional Requirements (NFR)

1. **Responsivitas Tampilan (Responsive Design)**:
   - Layout aplikasi harus dapat beradaptasi secara mulus pada layar ponsel (lebar 360px+), tablet, dan layar desktop (1920px).
2. **Kinerja Pemuatan Awal (First Contentful Paint)**:
   - Waktu FCP pada jaringan lokal harus < 1.0 detik dengan ukuran bundle JavaScript awal < 250 KB (gzipped).
3. **Pemisahan Logika & Tampilan (Separation of Concerns)**:
   - Seluruh logika komunikasi WebSocket dan REST API harus terisolasi di dalam folder layanan (`services/` atau `hooks/`) dan tidak dicampur ke dalam komponen tampilan murni.

---

## 3. Rujukan Dokumen
- [Design Fase 0 - Frontend Env](../00-project-bootstrap-environment/design.md#22-frontend-frontendenv)
- [Design Fase 4 - Kontrak REST API](../04-backend-rest-api/design.md#2-spesifikasi-endpoint--kontrak-respon)
- [Design Fase 5 - WebSocket Protocol](../05-backend-websocket-gateway/design.md#2-kontrak-frame-pesan-websocket)
