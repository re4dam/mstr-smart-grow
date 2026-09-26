# Daftar Tugas Implementasi (Tasks) — Fase 1: Database Schema & Migration

Daftar checklist tugas implementasi granular untuk skema basis data dan migrasi MariaDB. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Setup Perkakas Migrasi
- [ ] **TASK-101**: Pasang dependensi CLI atau library `github.com/golang-migrate/migrate/v4` beserta driver MySQL/MariaDB pada `backend/`.
- [ ] **TASK-102**: Buat runner migrasi di internal Go (`internal/database/migration.go`) yang mampu mengeksekusi migrasi otomatis saat backend start (opsional via flag `--migrate`).

## 2. Pembuatan Berkas Skema DDL SQL
- [ ] **TASK-103**: Buat file migrasi `000001_create_devices_table.up.sql` dan `down.sql` untuk tabel `devices`.
- [ ] **TASK-104**: Buat file migrasi `000002_create_readings_table.up.sql` dan `down.sql` untuk tabel `readings` lengkap dengan foreign key dan indeks komposit `idx_readings_device_timestamp`.
- [ ] **TASK-105**: Buat file migrasi `000003_create_analytics_snapshots_table.up.sql` dan `down.sql` untuk tabel `analytics_snapshots` dengan constraint unik `(device_id, period_type, period_date)`.

## 3. Seed Data & Verifikasi
- [ ] **TASK-106**: Buat skrip SQL seed data `backend/migrations/seeds/001_sample_pot_data.sql` berisi 1 perangkat dummy (`pot-01`, nama: "Monstera Deliciosa", lokasi: "Living Room") dan 100 baris data telemetri historis dengan timestamp realistis.
- [ ] **TASK-107**: Uji coba perintah migrasi: eksekusi migrasi penuh (*migrate up*), verifikasi skema tabel terbentuk di MariaDB Docker lokal.
- [ ] **TASK-108**: Uji coba rollback: eksekusi *migrate down*, verifikasi tabel terhapus bersih tanpa error relasi, kemudian eksekusi ulang *migrate up* dan muat data seed.

---

## Rujukan Dokumen
- [Requirements Fase 1](requirements.md)
- [Design Fase 1](design.md)
