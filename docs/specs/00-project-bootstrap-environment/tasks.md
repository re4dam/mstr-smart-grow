# Daftar Tugas Implementasi (Tasks) — Fase 0: Project Bootstrap & Environment

Daftar checklist tugas implementasi granular untuk inisialisasi lingkungan dan proyek. Setiap tugas dirancang untuk dikerjakan dalam lingkup 1 commit atau Pull Request (PR).

---

## 1. Setup Direktori & Git Configuration
- [ ] **TASK-001**: Buat struktur direktori dasar proyek (`backend/`, `frontend/`, `docs/specs/`).
- [ ] **TASK-002**: Buat file `.gitignore` di root repositori untuk mengabaikan direktori vendor, file `.env`, node_modules, binary build, dan volume data Docker.

## 2. Setup Modul Backend (Go)
- [ ] **TASK-003**: Inisialisasi Go module di `backend/` menggunakan perintah `go mod init`.
- [ ] **TASK-004**: Buat modul `internal/config/config.go` untuk memuat dan memvalidasi variabel lingkungan dengan fallback nilai default.
- [ ] **TASK-005**: Buat file `backend/.env.example` yang mencakup semua parameter MQTT, MariaDB, dan port server.
- [ ] **TASK-006**: Buat entrypoint minimal `backend/cmd/server/main.go` yang memuat konfigurasi dan memverifikasi startup log.
- [ ] **TASK-007**: Buat konfigurasi `.golangci.yml` dan script verifikasi linter pada `backend/`.

## 3. Setup Aplikasi Frontend (React Router)
- [ ] **TASK-008**: Inisialisasi proyek React Router / Vite TypeScript di folder `frontend/`.
- [ ] **TASK-009**: Buat file `frontend/.env.example` berisi variabel `VITE_API_BASE_URL` dan `VITE_WS_URL`.
- [ ] **TASK-010**: Konfigurasi ESLint (`.eslintrc.json`) dan Prettier (`.prettierrc`) di folder `frontend/`.
- [ ] **TASK-011**: Tambahkan npm scripts di `package.json` untuk `dev`, `build`, `lint`, dan `format`.

## 4. Orkestrasi Lingkungan Lokal (Docker Compose)
- [ ] **TASK-012**: Buat file `docker-compose.yml` di root untuk service MariaDB (port 3306) lengkap dengan volume persisten dan healthcheck.
- [ ] **TASK-013**: Tambahkan service MQTT broker Eclipse Mosquitto (image `eclipse-mosquitto:2` pada port 1883 beserta volume konfigurasi `mosquitto.conf`) ke dalam `docker-compose.yml`.
- [ ] **TASK-014**: Buat file `.env.example` di root repositori yang merangkum variabel untuk Docker Compose.
- [ ] **TASK-015**: Verifikasi eksekusi end-to-end: jalankan `docker compose up -d`, verifikasi MariaDB dan broker siap menerima koneksi jaringan.

---

## Rujukan Dokumen
- [Requirements Fase 0](requirements.md)
- [Design Fase 0](design.md)
