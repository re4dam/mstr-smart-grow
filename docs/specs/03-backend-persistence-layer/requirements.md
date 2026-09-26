# Spesifikasi Kebutuhan (Requirements) — Fase 3: Backend: Persistence Layer

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk layer persistensi database (*Repository / Data Access Object*), pipeline penulisan data telemetri dari MQTT Consumer ke MariaDB, strategi *worker pool/channel buffering*, serta penanganan retensi data.

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-301: Penyisipan Telemetri Secara Asinkron (Decoupled Ingestion)
Sebagai *sistem backend*, saya ingin proses penyimpanan database tidak memblokir goroutine callback MQTT subscriber agar pesan telemetri berikutnya tidak tertunda saat database lambat merespons.

- **REQ-301-01 (Ubiquitous)**: THE SYSTEM SHALL menyalurkan payload telemetri yang telah tervalidasi dari MQTT consumer ke layer persistensi melalui Go buffered channel.
- **REQ-301-02 (Event-driven)**: WHEN payload masuk ke dalam channel antrean persistensi, THE SYSTEM SHALL memproses penyimpanan ke tabel `readings` secara asinkron menggunakan *worker pool* berkapasitas terkontrol.
- **REQ-301-03 (State-driven)**: WHILE volume pesan tinggi dan channel antrean mencapai 80% kapasitas, THE SYSTEM SHALL mencatat log metrik peringatan (*warning*) terkait degradasi *throughput*.
- **REQ-301-04 (Unwanted event)**: IF antrean persistensi penuh (*channel buffer overflow*), THEN THE SYSTEM SHALL menerapkan strategi *drop oldest* atau menolak pesan secara terkontrol tanpa mematikan proses utama service.

### US-302: Penyimpanan Rekaman Sensor ke Tabel `readings`
Sebagai *database repository*, saya ingin menyimpan data pembacaan sensor dan aktuator secara konsisten ke tabel `readings` sesuai skema yang telah ditentukan pada Fase 1.

- **REQ-302-01 (Event-driven)**: WHEN payload telemetri baru diterima oleh worker persistensi, THE SYSTEM SHALL mengeksekusi perintah SQL `INSERT INTO readings` dalam waktu kurang dari 50ms.
- **REQ-302-02 (Unwanted event)**: IF koneksi database terputus sesaat saat operasi insert berlangsung, THEN THE SYSTEM SHALL mencoba mengulang operasi (*retry*) hingga 3 kali dengan jeda eksponensial sebelum membuang atau mencatat pesan ke log error.

### US-303: Pengambilan Rekaman Historis & Terkini
Sebagai *layer repository*, saya ingin menyediakan antarmuka kueri untuk mengambil rekaman telemetri terkini (*latest*) dan deret data historis berdasar rentang waktu untuk melayani kebutuhan REST API pada Fase 4.

- **REQ-303-01 (Event-driven)**: WHEN kueri `GetLatestReading(deviceID)` dipanggil, THE SYSTEM SHALL mengembalikan rekaman telemetri paling baru untuk perangkat tersebut dengan memanfaatkan indeks timestamp.
- **REQ-303-02 (Event-driven)**: WHEN kueri `GetHistoricalReadings(deviceID, fromTimestamp, toTimestamp, limit)` dipanggil, THE SYSTEM SHALL mengembalikan kumpulan rekaman sensor yang diurutkan secara kronologis (atau terbalik) sesuai parameter permintaan.

---

## 2. Non-Functional Requirements (NFR)

1. **Efisiensi Koneksi Database (Connection Pooling)**:
   - Repository harus mengelola *connection pool* MariaDB secara efisien (`SetMaxOpenConns(25)`, `SetMaxIdleConns(10)`, `SetConnMaxLifetime(5 * time.Minute)`).
2. **Latensi Penulisan (Insert Latency)**:
   - 95% dari operasi insert tunggal (*p95 write latency*) harus selesai dalam waktu < 20ms pada koneksi lokal.
3. **Keandalan Data (Data Reliability)**:
   - Tingkat kegagalan penyimpanan data akibat bug aplikasi (*internal error*) harus < 0.01% pada kondisi operasional normal.
4. **Graceful Worker Drain**:
   - Saat aplikasi menerima sinyal stop, antrean channel persistensi harus dikosongkan (*drained*) terlebih dahulu hingga selesai sebelum koneksi database ditutup (*flush on shutdown*).

---

## 3. Rujukan Dokumen
- [Design Fase 1 - Skema Tabel Readings](../01-database-schema-migration/design.md#22-tabel-readings)
- [Design Fase 2 - Struct TelemetryPayload](../02-backend-mqtt-consumer/design.md#1-struktur-struct-go-payload-telemetri)
- [Techstack Architecture - Flow Diagram Pola 1](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L25-L68)
