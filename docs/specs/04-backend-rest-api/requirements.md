# Spesifikasi Kebutuhan (Requirements) — Fase 4: Backend: REST API

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk penyediaan antarmuka REST API HTTP pada backend Go, mencakup endpoint data pembacaan telemetri terkini, data historis deret waktu, kalkulasi ringkasan analitik (penghematan kWh dan insight biologis), serta endpoint pemeriksaan kesehatan sistem (*health check*).

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-401: Pembacaan Telemetri Terkini
Sebagai *klien dashboard*, saya ingin mengambil data pembacaan sensor dan status aktuator paling mutakhir dari suatu pot agar dashboard dapat menampilkan kondisi awal saat pertama kali dimuat.

- **REQ-401-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan endpoint `GET /api/readings/latest` yang menerima parameter query opsional `device_id`.
- **REQ-401-02 (Event-driven)**: WHEN permintaan `GET /api/readings/latest` diterima tanpa `device_id`, THE SYSTEM SHALL mengembalikan data telemetri terkini dari perangkat aktif pertama yang terdaftar.
- **REQ-401-03 (Unwanted event)**: IF `device_id` yang diminta tidak ditemukan atau belum memiliki catatan telemetri, THEN THE SYSTEM SHALL mengembalikan kode status HTTP `404 Not Found` dengan pesan error terstruktur.

### US-402: Pengambilan Data Historis Time-Series
Sebagai *klien dashboard*, saya ingin mengambil riwayat data sensor berdasarkan rentang waktu tertentu agar dapat merender grafik tren kondisi lingkungan tanaman.

- **REQ-402-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan endpoint `GET /api/readings` dengan parameter query wajib `device_id`, serta parameter opsional `range` (`1h`, `6h`, `24h`, `7d`, `30d`, default: `24h`) dan `limit` (default: 100, max: 1000).
- **REQ-402-02 (Event-driven)**: WHEN permintaan `GET /api/readings` berhasil diproses, THE SYSTEM SHALL mengembalikan array data sensor yang diurutkan secara kronologis beserta metadata `count` dan `range`.
- **REQ-402-03 (Unwanted event)**: IF parameter query `device_id` tidak disertakan, THEN THE SYSTEM SHALL mengembalikan kode status HTTP `400 Bad Request`.

### US-403: Ringkasan Analitik Energi & Biologis
Sebagai *pengguna*, saya ingin melihat estimasi penghematan energi listrik (kWh) dan rekomendasi kesehatan biologis tanaman pada periode waktu tertentu agar saya mengetahui efisiensi sistem pencahayaan pintar.

- **REQ-403-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan endpoint `GET /api/analytics/summary` dengan parameter query wajib `device_id` dan opsional `period` (`today`, `this_week`, `this_month`).
- **REQ-403-02 (Event-driven)**: WHEN endpoint dipanggil, THE SYSTEM SHALL menghitung:
  - *Baseline consumption* (asumsi lampu berdaya nominal 15W menyala 100% penuh selama jadwal siklus terang).
  - *Actual consumption* (integrasi daya riil berdasarkan rata-rata persentase PWM dimming dikalikan daya nominal).
  - *kWh savings* dan *percentage savings*.
  - Skor kesehatan tanah (0-100) dan skor kecukupan cahaya (0-100).
  - Daftar teks rekomendasi perawatan tanaman yang dapat ditindaklanjuti (*actionable recommendations*).

### US-404: Pemeriksaan Kesehatan Sistem (Health Check)
Sebagai *sistem monitoring / DevOps*, saya ingin memeriksa kesiapan operasional backend dan konektivitas dependensi eksternal (MQTT & Database).

- **REQ-404-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan endpoint `GET /api/health`.
- **REQ-404-02 (State-driven)**: WHILE seluruh koneksi (broker MQTT dan database MariaDB) aktif, THE SYSTEM SHALL mengembalikan status HTTP `200 OK` dengan payload `{"status":"ok", "mqtt_connected":true, "database_connected":true}`.
- **REQ-404-03 (Unwanted event)**: IF koneksi database atau broker terputus, THEN THE SYSTEM SHALL mengembalikan status HTTP `503 Service Unavailable` beserta status status komponen yang terdampak.

---

## 2. Non-Functional Requirements (NFR)

1. **Waktu Respon (Response Latency)**:
   - 95% permintaan kueri `GET /api/readings/latest` harus selesai dalam waktu < 25ms.
   - 95% permintaan kueri agregasi historis `GET /api/readings` (rentang 24 jam) harus selesai dalam waktu < 100ms.
2. **Standardisasi Format Error**:
   - Seluruh respon error (4xx dan 5xx) harus menggunakan skema JSON terpadu: `{"error": {"code": "...", "message": "...", "details": ...}}`.
3. **CORS (Cross-Origin Resource Sharing)**:
   - Server REST API harus mengizinkan header CORS untuk origin dashboard frontend lokal (`http://localhost:5173` atau variabel `FRONTEND_URL`).
4. **Idempotensi**:
   - Seluruh endpoint GET harus bersifat aman (*safe*) dan idempotent tanpa mengubah status penyimpanan data.

---

## 3. Rujukan Dokumen
- [Techstack Architecture - REST API Endpoints](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L120-L205)
- [Design Fase 1 - Skema MariaDB](../01-database-schema-migration/design.md#2-definisi-skema-ddl-sql)
- [Design Fase 3 - Interface Repository](../03-backend-persistence-layer/design.md#3-kontrak-go-repository-interface)
