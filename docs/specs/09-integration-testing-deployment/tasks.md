# Daftar Tugas Implementasi (Tasks) — Fase 9: Integration Testing & Deployment

Daftar checklist tugas implementasi granular untuk pengujian integrasi ujung-ke-ujung dan deployment produksi. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Skrip Simulator & Pengujian E2E
- [ ] **TASK-901**: Buat skrip simulasi perangkat ESP32 (`scripts/simulate_pot.py` atau `scripts/simulate_pot.go`) yang memublikasikan payload telemetri dengan variasi lux dan kelembapan dinamis ke broker MQTT.
- [ ] **TASK-902**: Buat test runner E2E otomatis (`tests/e2e_test.go`) yang memverifikasi aliran: publikasi simulator → persistensi di MariaDB → penerimaan frame WebSocket di klien uji → verifikasi respon `GET /api/readings/latest`.

## 2. Kontainerisasi & Konfigurasi Server Produksi
- [ ] **TASK-903**: Buat file `backend/Dockerfile` multi-stage build yang menghasilkan binary Go statis berbasis image minimal Alpine.
- [ ] **TASK-904**: Buat file `frontend/Dockerfile` multi-stage dan `frontend/nginx.conf` untuk menyajikan build statis React Router dengan konfigurasi SPA fallback routing.
- [ ] **TASK-905**: Buat file `docker-compose.prod.yml` yang mengorkestrasikan backend, frontend web server, dan MariaDB dengan network bridge terisolasi dan restart policy `unless-stopped`.

## 3. Standardisasi Logging & CI Pipeline
- [ ] **TASK-906**: Konfigurasi logger `log/slog` pada backend agar memancarkan format JSON terstruktur saat `APP_ENV=production`.
- [ ] **TASK-907**: Buat workflow GitHub Actions CI (`.github/workflows/ci.yml`) yang menjalankan unit test Go, linter Go, build frontend, dan validasi build Docker.
- [ ] **TASK-908**: Lakukan verifikasi deployment lokal produksi: jalankan `docker compose -f docker-compose.prod.yml up -d` dan uji coba fungsionalitas end-to-end dashboard di browser.

---

## Rujukan Dokumen
- [Requirements Fase 9](requirements.md)
- [Design Fase 9](design.md)
