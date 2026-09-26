# Daftar Tugas Implementasi (Tasks) — Fase 2: Backend: MQTT Consumer

Daftar checklist tugas implementasi granular untuk modul MQTT Consumer. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Dependensi & Struktur Modul
- [ ] **TASK-201**: Tambahkan pustaka MQTT `github.com/eclipse/paho.mqtt.golang` ke `backend/go.mod`.
- [ ] **TASK-202**: Buat direktori `backend/internal/mqtt/` dan definisikan struct data payload telemetri (`payload.go`).

## 2. Parsing & Logika Validasi
- [ ] **TASK-203**: Implementasikan fungsi deserialisasi JSON `ParsePayload(raw []byte) (*TelemetryPayload, error)`.
- [ ] **TASK-204**: Implementasikan fungsi validasi rentang fisik `Validate(p *TelemetryPayload) error` untuk semua metrik sensor dan aktuator.
- [ ] **TASK-205**: Buat unit test komprehensif (`payload_test.go`) yang menguji:
  - Payload valid standar.
  - JSON malformed / sintaks salah.
  - Nilai sensor di luar batas ambang (lux negatif, suhu > 85°C, kelembapan > 100%).
  - Fallback ekstraksi `device_id` dari topik bila payload kosong.

## 3. Koneksi Broker & Subscription
- [ ] **TASK-206**: Buat service client MQTT `internal/mqtt/client.go` yang mengatur koneksi TLS/TCP, reconnect backoff, dan handler `OnConnect` / `OnConnectionLost`.
- [ ] **TASK-207**: Implementasikan fungsi subscription `SubscribeTelemetry(topic string, handler MessageHandler)` dengan opsi QoS 1.
- [ ] **TASK-208**: Hubungkan MQTT Consumer ke `backend/cmd/server/main.go` lengkap dengan graceful shutdown via context cancellation.
- [ ] **TASK-209**: Lakukan uji coba koneksi lokal: jalankan broker via Docker Compose, publish contoh pesan MQTT menggunakan CLI `mosquitto_pub` atau skrip Go, dan verifikasi log parsing berhasil.

---

## Rujukan Dokumen
- [Requirements Fase 2](requirements.md)
- [Design Fase 2](design.md)
