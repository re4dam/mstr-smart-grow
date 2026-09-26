# Daftar Tugas Implementasi (Tasks) — Fase 4: Backend: REST API

Daftar checklist tugas implementasi granular untuk layer REST API backend. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Setup Server HTTP & Middleware
- [ ] **TASK-401**: Buat package `internal/api/` dan router HTTP terpusat menggunakan router Go 1.22+ (`http.NewServeMux`).
- [ ] **TASK-402**: Implementasikan middleware CORS (`internal/api/middleware/cors.go`) yang mengizinkan origin frontend lokal.
- [ ] **TASK-403**: Implementasikan middleware structured request logging menggunakan `log/slog` dan middleware penanganan panic/recovery.
- [ ] **TASK-404**: Buat helper format JSON respon terpadu (`internal/api/response.go`) untuk respon sukses dan format error seragam.

## 2. Implementasi Endpoint & Business Logic
- [ ] **TASK-405**: Implementasikan handler `GET /api/readings/latest` yang mengambil pembacaan terakhir dari repository dan memvalidasi `device_id`.
- [ ] **TASK-406**: Implementasikan handler `GET /api/readings` yang mem-parsing parameter `range` (`1h`, `6h`, `24h`, dll.) dan `limit` menjadi batas timestamp kueri database.
- [ ] **TASK-407**: Buat package mesin analitik `internal/analytics/engine.go` yang menghitung konsumsi baseline vs aktual (kWh) dan skor biologis.
- [ ] **TASK-408**: Implementasikan handler `GET /api/analytics/summary` yang mengintegrasikan mesin analitik dengan data historis.
- [ ] **TASK-409**: Implementasikan handler `GET /api/health` yang mengecek status ping konektivitas MariaDB dan broker MQTT.

## 3. Pengujian Integrasi API
- [ ] **TASK-410**: Buat test suite integrasi HTTP (`api_test.go`) menggunakan `httptest.NewServer` atau `httptest.ResponseRecorder` untuk memvalidasi:
  - Respon 200 OK dan struktur JSON dari `GET /api/readings/latest`.
  - Respon 400 Bad Request jika `device_id` tidak disertakan pada `GET /api/readings`.
  - Respon 404 Not Found untuk `device_id` yang tidak terdaftar.
  - Perhitungan matematis penghematan energi pada `GET /api/analytics/summary`.

---

## Rujukan Dokumen
- [Requirements Fase 4](requirements.md)
- [Design Fase 4](design.md)
