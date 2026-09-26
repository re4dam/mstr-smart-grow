# Daftar Tugas Implementasi (Tasks) — Fase 3: Backend: Persistence Layer

Daftar checklist tugas implementasi granular untuk layer persistensi database MariaDB. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Koneksi Database & Pool Management
- [ ] **TASK-301**: Buat package `internal/database/mariadb.go` untuk inisialisasi koneksi `database/sql` ke MariaDB menggunakan konfigurasi dari Fase 0.
- [ ] **TASK-302**: Konfigurasi parameter pool koneksi (`SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`) dan buat fungsi `Ping(ctx)` untuk healthcheck.

## 2. Implementasi Repository Layer
- [ ] **TASK-303**: Buat package `internal/repository/` dan definisikan struct `ReadingRecord` serta interface `ReadingRepository`.
- [ ] **TASK-304**: Implementasikan fungsi `SaveReading(ctx, payload)` dengan kueri parameterized `INSERT INTO readings`.
- [ ] **TASK-305**: Implementasikan fungsi `GetLatestReading(ctx, deviceID)` yang memanfaatkan indeks komposit `idx_readings_device_timestamp`.
- [ ] **TASK-306**: Implementasikan fungsi `GetHistoricalReadings(ctx, deviceID, fromTs, toTs, limit)` dengan filter rentang waktu.

## 3. Worker Pool & Integrasi MQTT Consumer
- [ ] **TASK-307**: Buat antrean buffer channel `internal/worker/persistence_worker.go` dengan kapasitas yang dapat dikonfigurasi (mis. 1000 item).
- [ ] **TASK-308**: Implementasikan worker pool goroutine yang membaca dari channel antrean dan memanggil repository `SaveReading` dengan mekanisme retry jika terjadi gangguan jaringan sesaat.
- [ ] **TASK-309**: Hubungkan callback MQTT consumer dari Fase 2 ke antrean channel persistensi secara non-blocking.
- [ ] **TASK-310**: Implementasikan *graceful drain*: saat aplikasi shutdown, pastikan antrean channel tersisa diproses hingga kosong sebelum koneksi database ditutup.
- [ ] **TASK-311**: Buat pengujian integrasi (`repository_test.go`) dengan test database MariaDB untuk memverifikasi operasi insert, latest reading, dan time-range filter.

---

## Rujukan Dokumen
- [Requirements Fase 3](requirements.md)
- [Design Fase 3](design.md)
