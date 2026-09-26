# Daftar Tugas Implementasi (Tasks) — Fase 5: Backend: WebSocket Gateway

Daftar checklist tugas implementasi granular untuk modul WebSocket Gateway. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Setup Modul & Hub Concurrency
- [ ] **TASK-501**: Tambahkan pustaka WebSocket (`github.com/gorilla/websocket`) ke dependensi `backend/go.mod`.
- [ ] **TASK-502**: Buat package `internal/websocket/` dan implementasikan struct `Hub` beserta loop konkurensi goroutine untuk aksi `register`, `unregister`, dan `broadcast`.
- [ ] **TASK-503**: Buat struct `Client` dengan implementasi `readPump` dan `writePump` untuk komunikasi socket non-blocking.

## 2. Heartbeat & Error Handling
- [ ] **TASK-504**: Implementasikan mekanisme ping/pong timer berkala (ping setiap 30s, pong wait 60s) pada `Client`.
- [ ] **TASK-505**: Implementasikan deteksi *slow consumer*: putus koneksi dan buang buffer jika antrean `send` klien meluap.
- [ ] **TASK-506**: Implementasikan HTTP handler `ServeWS(hub *Hub, w http.ResponseWriter, r *http.Request)` yang melakukan upgrade protokol pada endpoint `/ws/live` dengan validasi origin CORS.

## 3. Integrasi Pipeline & Testing
- [ ] **TASK-507**: Sambungkan broadcast channel Hub ke callback MQTT consumer dari Fase 2 sehingga setiap payload yang masuk langsung diserialisasikan ke format frame WebSocket.
- [ ] **TASK-508**: Tambahkan endpoint `/ws/live` ke router utama server HTTP di `backend/cmd/server/main.go`.
- [ ] **TASK-509**: Implementasikan graceful shutdown: saat server dimatikan, kirim close frame standar ke seluruh klien aktif.
- [ ] **TASK-510**: Buat pengujian integrasi WebSocket (`ws_test.go`) yang menghubungkan klien dummy, menyimulasikan injeksi data MQTT, dan memverifikasi penerimaan frame event `telemetry_update`.

---

## Rujukan Dokumen
- [Requirements Fase 5](requirements.md)
- [Design Fase 5](design.md)
