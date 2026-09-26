# Spesifikasi Kebutuhan (Requirements) — Fase 5: Backend: WebSocket Gateway

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk modul WebSocket Gateway pada Go backend yang bertugas mengelola koneksi klien peramban (*browser clients*), menyiarkan data telemetri secara *real-time* (*broadcasting*), menangani siklus hidup koneksi (*heartbeat/ping-pong*), serta penghentian anggun (*graceful shutdown*).

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-501: Upgrade Koneksi HTTP ke WebSocket
Sebagai *klien dashboard*, saya ingin meng-upgrade koneksi HTTP ke WebSocket pada endpoint `/ws/live` agar dapat menerima aliran data telemetri secara berkelanjutan tanpa melakukan *HTTP polling*.

- **REQ-501-01 (Event-driven)**: WHEN klien mengirimkan permintaan HTTP GET dengan header `Upgrade: websocket` ke endpoint `/ws/live`, THE SYSTEM SHALL memvalidasi origin dan meng-upgrade koneksi ke protokol WebSocket (RFC 6455).
- **REQ-501-02 (Unwanted event)**: IF permintaan upgrade koneksi berasal dari origin yang tidak diizinkan atau header WebSocket tidak lengkap, THEN THE SYSTEM SHALL menolak permintaan dengan kode status HTTP `403 Forbidden` atau `400 Bad Request`.

### US-502: Penyiaran Telemetri Real-Time (Live Broadcast)
Sebagai *pengguna*, saya ingin setiap pembacaan sensor baru dari pot tanaman langsung dikirimkan ke layar dashboard dalam waktu sub-detik agar status tanaman dapat dipantau seketika.

- **REQ-502-01 (Event-driven)**: WHEN payload telemetri baru diterima dari MQTT Consumer (Fase 2), THE SYSTEM SHALL menyiarkan (*broadcast*) payload dalam format JSON event `telemetry_update` ke seluruh klien WebSocket yang sedang terhubung.
- **REQ-502-02 (State-driven)**: WHILE klien mengalami hambatan jaringan (*slow consumer* / buffer penuh), THE SYSTEM SHALL memutuskan koneksi klien yang macet tersebut secara aman tanpa mengganggu pengiriman data ke klien lain.

### US-503: Deteksi Keaktifan Koneksi (Heartbeat & Keepalive)
Sebagai *sistem backend*, saya ingin mendeteksi koneksi klien yang mati tanpa pemutusan formal (*ghost connection / half-open*) agar sumber daya memori server tidak terbuang sia-sia.

- **REQ-503-01 (Ubiquitous)**: THE SYSTEM SHALL mengirimkan pesan ping secara periodik (setiap 30 detik) ke setiap klien yang terhubung.
- **REQ-503-02 (Unwanted event)**: IF klien tidak membalas frame pong dalam batas waktu 60 detik (*pong wait timeout*), THEN THE SYSTEM SHALL menutup koneksi klien tersebut dan menghapusnya dari daftar registrasi aktif.

### US-504: Manajemen Banyak Klien & Graceful Shutdown
Sebagai *sistem backend*, saya ingin mengelola pendaftaran dan pelepasan klien secara *thread-safe* serta menutup seluruh koneksi klien secara tertib saat server dimatikan.

- **REQ-504-01 (Ubiquitous)**: THE SYSTEM SHALL mengelola siklus hidup koneksi (register, unregister, broadcast) menggunakan model *hub concurrency* berbasis Go channel atau sync mutex.
- **REQ-504-02 (Event-driven)**: WHEN sinyal shutdown diterima (`SIGINT`/`SIGTERM`), THE SYSTEM SHALL mengirimkan frame penutupan WebSocket (close code 1001 Going Away) ke seluruh klien sebelum server berhenti.

---

## 2. Non-Functional Requirements (NFR)

1. **Latensi Penyiaran (Broadcast Latency)**:
   - Jeda waktu antara penerimaan data di backend dari MQTT hingga terkirimnya frame WebSocket ke klien lokal harus < 10 milidetik.
2. **Kapasitas Konkurensi Klien (Concurrent Clients)**:
   - Gateway harus mampu melayani minimal 100 koneksi WebSocket konkuren pada memori < 50MB.
3. **Isolasi Kegagalan (Fault Isolation)**:
   - Kegagalan pengiriman pesan pada satu koneksi klien tidak boleh memblokir atau memperlambat pengiriman ke koneksi klien lainnya.

---

## 3. Rujukan Dokumen
- [Techstack Architecture - WebSocket Gateway](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L208-L238)
- [Design Fase 2 - Struct TelemetryPayload](../02-backend-mqtt-consumer/design.md#1-struktur-struct-go-payload-telemetri)
- [Glosarium Istilah - WebSocket Gateway](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/glossary.md#L50)
