# Smart Grow Pot — Technical Documentation

## Ringkasan Proyek
Smart Grow Pot adalah sistem pot tanaman cerdas berbasis Internet of Things (IoT) yang menggabungkan pemantauan kondisi lingkungan (cahaya, suhu, kelembapan udara, kelembapan tanah) dengan sistem pencahayaan buatan adaptif (*adaptive grow lighting*). Proyek ini bertujuan untuk menyediakan kondisi pertumbuhan optimal bagi tanaman di dalam ruangan (*indoor*) secara otomatis, mencegah kekurangan atau kelebihan paparan cahaya serta hidrasi, sekaligus menekan konsumsi energi listrik secara efisien melalui mekanisme peredupan cahaya pintar (*dimming logic*).

---

## Panduan Menjalankan Dashboard

Arsitektur aplikasi web mengadopsi pemisahan antara service backend berbasis Go dan frontend berbasis React Router.

### 1. Menjalankan Backend Service (Go)

Backend berfungsi sebagai MQTT consumer, persistence worker ke MariaDB, penyedia REST API data historis & analitik, serta WebSocket gateway untuk *live update*.

#### Prasyarat
- Go 1.22+ terinstal
- Akses ke Eclipse Mosquitto MQTT Broker (self-hosted via Docker atau service lokal/remote)
- Instance MariaDB yang aktif

#### Variabel Lingkungan (*Environment Variables*)
Siapkan file `.env` di dalam direktori `backend/` atau *export* variabel berikut:

```bash
# Konfigurasi MQTT Broker (Mosquitto)
MQTT_BROKER_URL=tcp://localhost:1883
MQTT_CLIENT_ID=smartgrow-backend-consumer
MQTT_USERNAME=your_username
MQTT_PASSWORD=your_password
MQTT_TOPIC=smartgrow/+/telemetry

# Konfigurasi Database (MariaDB)
MARIADB_HOST=localhost
MARIADB_PORT=3306
MARIADB_USER=smartgrow_user
MARIADB_PASSWORD=smartgrow_pass
MARIADB_DATABASE=smartgrow_db

# Konfigurasi Server HTTP & WebSocket
PORT=8080
```

#### Menjalankan Service
```bash
# Masuk ke direktori backend (TODO: sesuaikan path subfolder bila sudah dibuat)
cd backend

# Unduh dependensi
go mod download

# Jalankan dalam mode pengembangan
go run main.go

# Atau build binary production
go build -o smartgrow-backend main.go
./smartgrow-backend
```

---

### 2. Menjalankan Frontend Dashboard (React Router)

Frontend bertanggung jawab menampilkan data pemantauan real-time via WebSocket serta visualisasi grafik telemetri historis dan estimasi penghematan energi via REST API.

#### Prasyarat
- Node.js 18+ atau 20+
- npm / pnpm / yarn

#### Menjalankan Dev Server
```bash
# Masuk ke direktori frontend (TODO: sesuaikan path subfolder bila sudah dibuat)
cd frontend

# Pasang dependensi
npm install

# Konfigurasi endpoint backend jika diperlukan
# Misal: VITE_API_BASE_URL=http://localhost:8080/api
#        VITE_WS_URL=ws://localhost:8080/ws

# Jalankan server pengembangan
npm run dev
```

---

## Struktur Folder Monorepo (Tingkat Tinggi)

```
.
├── backend/                  # Service backend Go (MQTT Consumer, REST API, WebSocket Gateway)
│   ├── cmd/                  # Entrypoint aplikasi (TODO: konfirmasi struktur package)
│   ├── internal/             # Logika internal (mqtt, database, handlers, websocket)
│   ├── go.mod
│   └── go.sum
├── frontend/                 # Web dashboard berbasis React Router
│   ├── app/                  # Routing, routes, dan komponen halaman (React Router)
│   ├── package.json
│   └── vite.config.ts
├── firmware/                 # Kode sumber Arduino/ESP32 (TODO: struktur final modul firmware)
│   └── smart_grow_pot.ino
└── docs/                     # Dokumentasi teknis terpusat
    ├── README.md             # Dokumen panduan utama ini
    ├── overview.md           # Narasi end-to-end arsitektur dan tanggung jawab tim
    ├── glossary.md           # Glosarium istilah domain & teknis
    ├── CHANGELOG.md          # Log perubahan berbasis Keep a Changelog
    ├── change-requests-log.md# Tabel pelacakan usulan perubahan (Change Request)
    ├── architecture/
    │   └── Techstack.md      # Detail teknologi, diagram data, payload, dan API
    └── specs/                # Dokumen spesifikasi SDD bertahap (Fase 0 s.d. 9)
```

---

## Navigasi Dokumentasi Teknis

Untuk pemahaman mendalam tentang arsitektur dan spesifikasi proyek, silakan rujuk dokumen-dokumen berikut:
- [docs/overview.md](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/overview.md) — Penjelasan arsitektur menyeluruh dari sensor hingga dashboard, diagram alur sistem, serta batas tanggung jawab kerja tim.
- [docs/architecture/Techstack.md](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md) — Matriks teknologi, diagram alur data Mermaid, skema payload MQTT JSON, spesifikasi REST API, dan event WebSocket.
- [docs/glossary.md](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/glossary.md) — Kamus istilah domain agrikultur/lighting dan istilah teknis perangkat lunak.
- [docs/specs/](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/specs/) — Spesifikasi implementasi bertahap (*Spec-Driven Development*) Fase 0 hingga Fase 9 lengkap dengan EARS requirements, technical design, dan checklist tasks.
- [docs/CHANGELOG.md](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/CHANGELOG.md) — Riwayat revisi dan status rilis fitur.
- [docs/change-requests-log.md](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/change-requests-log.md) — Log pengajuan perubahan arsitektural dan spesifikasi sistem.
