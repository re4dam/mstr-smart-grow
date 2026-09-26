# Spesifikasi Kebutuhan (Requirements) — Fase 0: Project Bootstrap & Environment

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk inisialisasi lingkungan pengembangan, struktur repositori monorepo, konfigurasi variabel lingkungan (*environment variables*), kontainerisasi lokal (*Docker Compose*), serta standardisasi kualitas kode (*linting/formatting*).

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-001: Struktur Repositori Monorepo
Sebagai *developer*, saya ingin repositori memiliki struktur monorepo yang rapi untuk backend Go dan frontend React Router agar kode tersusun secara modular dan terisolasi.

- **REQ-001-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan direktori `backend/` untuk modul Go dan direktori `frontend/` untuk aplikasi React Router.
- **REQ-001-02 (Event-driven)**: WHEN perintah `go mod tidy` dijalankan pada direktori `backend/`, THE SYSTEM SHALL mengunduh seluruh dependensi Go tanpa error.
- **REQ-001-03 (Event-driven)**: WHEN perintah `npm install` dijalankan pada direktori `frontend/`, THE SYSTEM SHALL menyelesaikan instalasi dependensi tanpa dependensi siklik atau peringatan kritis.

### US-002: Manajemen Konfigurasi & Variabel Lingkungan
Sebagai *developer*, saya ingin mengonfigurasi kredensial broker MQTT, database MariaDB, dan port server melalui variabel lingkungan agar kredensial tidak bocor ke dalam version control.

- **REQ-002-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan file `.env.example` pada direktori root, `backend/`, dan `frontend/` yang memuat seluruh variabel wajib beserta nilai contoh.
- **REQ-002-02 (State-driven)**: WHILE variabel lingkungan yang wajib (*mandatory*) tidak terdefinisi saat startup backend, THE SYSTEM SHALL menolak start (*fail-fast*) dan mengeluarkan pesan log error deskriptif yang mengidentifikasi variabel yang hilang.
- **REQ-002-03 (Unwanted event)**: IF format nilai variabel lingkungan (misalnya port bukan angka numerik) tidak valid, THEN THE SYSTEM SHALL menghentikan proses inisialisasi dengan kode keluar (*exit code*) non-nol.

### US-003: Lingkungan Pengembangan Lokal (Local Stack)
Sebagai *developer*, saya ingin menjalankan MariaDB dan HiveMQ broker lokal menggunakan satu perintah container agar proses *onboarding* dan *testing* lokal tidak memerlukan instalasi manual.

- **REQ-003-01 (Event-driven)**: WHEN perintah `docker compose up -d` dijalankan, THE SYSTEM SHALL menjalankan kontainer MariaDB dan broker MQTT lokal dalam jaringan virtual (*bridge network*) yang sama.
- **REQ-003-02 (Event-driven)**: WHEN kontainer MariaDB berjalan, THE SYSTEM SHALL menyediakan mekanisme *healthcheck* yang mengonfirmasi database siap menerima koneksi pada port 3306 sebelum backend mencoba terhubung.
- **REQ-003-03 (Optional feature)**: WHERE pengembang tidak memiliki akses ke kluster HiveMQ Cloud, THE SYSTEM SHALL mendukung pengalihan ke broker MQTT lokal (HiveMQ CE atau Eclipse Mosquitto) melalui konfigurasi variabel lingkungan.

### US-004: Standardisasi Linter & Formatter Kode
Sebagai *developer*, saya ingin adanya aturan linting dan pemformatan otomatis untuk Go dan TypeScript/React agar basis kode tetap konsisten dan bebas dari potensi bug dasar.

- **REQ-004-01 (Event-driven)**: WHEN linter `golangci-lint run` dijalankan di direktori `backend/`, THE SYSTEM SHALL memvalidasi kepatuhan terhadap aturan `gofmt`, `govet`, `errcheck`, dan `staticcheck`.
- **REQ-004-02 (Event-driven)**: WHEN script linting dijalankan di direktori `frontend/`, THE SYSTEM SHALL mengecek sintaks TypeScript dan aturan hooks React menggunakan ESLint dan Prettier.

---

## 2. Non-Functional Requirements (NFR)

1. **Portabilitas Lingkungan (Portability)**:
   - File `docker-compose.yml` harus dapat dijalankan pada lingkungan Linux, macOS, dan Windows (WSL2) tanpa modifikasi manual.
2. **Waktu Startup Lokal (Developer Experience)**:
   - Waktu inisialisasi container lokal dari eksekusi `docker compose up` hingga status *healthy* tidak boleh melebihi 30 detik pada mesin standar (4-core, 8GB RAM).
3. **Keamanan Kredensial (Security)**:
   - File `.env` aktual harus dimasukkan ke dalam `.gitignore` utama agar tidak pernah ter-commit ke repositori git publik maupun privat.
4. **Fail-Fast Configuration**:
   - Backend harus memvalidasi seluruh konfigurasi saat inisialisasi awal (< 500ms) sebelum membuka koneksi jaringan ke database atau broker.

---

## 3. Rujukan Dokumen
- [README Proyek](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/README.md#L21-L42) — Variabel lingkungan dan panduan menjalankan service.
- [Techstack Overview](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L7-L18) — Matriks tumpukan teknologi backend dan frontend.
