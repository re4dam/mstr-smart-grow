# Spesifikasi Kebutuhan (Requirements) — Fase 1: Database Schema & Migration

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk perancangan skema database MariaDB, manajemen migrasi skema berbasis kode (*database migration*), indeks kueri *time-series*, serta data awal (*seed data*) untuk pengujian lokal.

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-101: Registrasi Perangkat Pot (Multi-Pot Management)
Sebagai *sistem backend*, saya ingin mencatat metadata perangkat pot pintar yang terdaftar ke dalam tabel `devices` agar telemetri yang masuk memiliki relasi identitas fisik yang valid.

- **REQ-101-01 (Ubiquitous)**: THE SYSTEM SHALL menyimpan metadata identitas setiap pot (identifier unik, nama pot, lokasi penempatan, dan status aktif) pada tabel `devices`.
- **REQ-101-02 (Event-driven)**: WHEN perangkat pot baru didaftarkan atau pertama kali mengirimkan data telemetri yang valid, THE SYSTEM SHALL menjamin integritas referensial antara entitas pembacaan sensor dan ID pot.
- **REQ-101-03 (Unwanted event)**: IF pendaftaran perangkat menggunakan `device_id` yang telah terdaftar, THEN THE SYSTEM SHALL memperbarui timestamp `updated_at` tanpa menggandakan rekaman kunci utama (*primary key*).

### US-102: Penyimpanan Pembacaan Sensor Time-Series
Sebagai *persistence worker*, saya ingin menyimpan data telemetri historis dari sensor dan aktuator ke dalam tabel `readings` agar dapat dianalisis dan divisualisasikan dalam grafik dashboard.

- **REQ-102-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan tabel `readings` yang memuat kolom: `id`, `device_id`, `timestamp` (Unix epoch detik), `light_lux`, `temperature_c`, `humidity_pct`, `soil_moisture_pct`, `grow_light_dimming_pct`, `soil_condition`, `lighting_mode`, dan `created_at`.
- **REQ-102-02 (Event-driven)**: WHEN satu baris data telemetri baru disisipkan (*insert*), THE SYSTEM SHALL menyelesaikan operasi penulisan dalam waktu kurang dari 50 milidetik pada kondisi beban normal.
- **REQ-102-03 (State-driven)**: WHILE kueri filter rentang waktu dijalankan dengan kombinasi `device_id` dan `timestamp`, THE SYSTEM SHALL memanfaatkan indeks komposit (`device_id`, `timestamp DESC`) untuk menghindari *full-table scan*.

### US-103: Snapshot Hasil Kalkulasi Analitik
Sebagai *mesin analitik*, saya ingin menyimpan agregat metrik harian/mingguan (penghematan energi kWh dan skor biologis) ke dalam tabel `analytics_snapshots` agar kueri dashboard tidak menghitung ulang data historis mentah secara berulang.

- **REQ-103-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan tabel `analytics_snapshots` untuk menyimpan metrik konsumsi energi (*baseline kWh*, *actual kWh*, *kwh savings*, *savings %*), skor kesehatan tanah, skor kecukupan cahaya, serta array rekomendasi teks dalam format JSON.
- **REQ-103-02 (Event-driven)**: WHEN snapshot analitik untuk periode tertentu telah dihitung, THE SYSTEM SHALL menyimpan atau memperbarui rekaman snapshot sesuai kombinasi `device_id`, `period_type`, dan `period_start`.

### US-104: Mekanisme Migrasi Skema & Seeding
Sebagai *developer*, saya ingin menjalankan migrasi skema dan memuat *seed data* secara otomatis melalui CLI agar skema database selalu sinkron di seluruh lingkungan pengembangan.

- **REQ-104-01 (Event-driven)**: WHEN perintah migrasi dieksekusi, THE SYSTEM SHALL menerapkan seluruh berkas migrasi versi secara berurutan dan mencatat versi terkini pada tabel skema migrasi.
- **REQ-104-02 (Event-driven)**: WHEN perintah rollback dieksekusi, THE SYSTEM SHALL mengembalikan perubahan skema ke versi sebelumnya tanpa merusak integritas tabel lain.
- **REQ-104-03 (Optional feature)**: WHERE flag *seed* diaktifkan, THE SYSTEM SHALL mengisi tabel `devices` dan tabel `readings` dengan 100+ sampel data historis untuk perangkat pengujian default `pot-01`.

---

## 2. Non-Functional Requirements (NFR)

1. **Performa Kueri (Query Latency)**:
   - Kueri rentang 24 jam terakhir untuk suatu `device_id` pada tabel `readings` dengan volume hingga 500.000 baris harus dieksekusi dalam waktu < 30ms berkat indeks komposit.
2. **Integritas & Presisi Data (Data Accuracy)**:
   - Nilai pembacaan floating-point (`light_lux`, `temperature_c`, dll.) harus disimpan dengan presisi minimal 2 angka di belakang koma (`FLOAT` atau `DECIMAL(6,2)`).
3. **Idempotensi Migrasi (Migration Idempotency)**:
   - Seluruh berkas migrasi SQL (up/down) harus bersifat terisolasi dan *atomic* menggunakan transaksi database bila didukung engine.
4. **Skalabilitas & Transisi (Future-Proofing)**:
   - Desain skema MariaDB harus memisahkan layer waktu dengan jelas agar memudahkan migrasi skema ke TimescaleDB (hypertable) sesuai pengajuan perubahan [CR-001](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/change-requests-log.md#L11).

---

## 3. Rujukan Dokumen
- [Techstack Architecture - Storage Layer](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L15)
- [Techstack Architecture - MQTT Payload Spec](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L82-L115)
- [Change Request Log CR-001](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/change-requests-log.md#L11)
