# Spesifikasi Kebutuhan (Requirements) — Fase 2: Backend: MQTT Consumer

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk modul MQTT Consumer pada Go backend yang bertugas menghubungkan service ke broker Eclipse Mosquitto, berlangganan ke topik telemetri ESP32, melakukan parsing payload JSON, memvalidasi integritas data, serta menangani skenario putus koneksi (*reconnect*).

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-201: Koneksi Persisten ke Broker Eclipse Mosquitto
Sebagai *sistem backend*, saya ingin mempertahankan koneksi persisten ke broker Eclipse Mosquitto (port 1883 default atau 8883 dengan TLS) agar telemetri dari mikrokontroler pot dapat diterima secara instan.

- **REQ-201-01 (Ubiquitous)**: THE SYSTEM SHALL membuat koneksi MQTT client menggunakan pustaka `paho.mqtt.golang` dengan konfigurasi Client ID yang unik dan kredensial yang diambil dari variabel lingkungan.
- **REQ-201-02 (State-driven)**: WHILE URL broker menggunakan skema `ssl://` atau port 8883, THE SYSTEM SHALL mengaktifkan enkripsi TLS 1.2+ dengan verifikasi sertifikat root CA standar.
- **REQ-201-03 (Unwanted event)**: IF koneksi MQTT ke broker terputus, THEN THE SYSTEM SHALL mengeksekusi mekanisme *exponential backoff reconnect* otomatis (mulai 1s hingga maksimal 60s) tanpa memicu crash (*panic*) pada aplikasi.

### US-202: Langganan Topik Telemetri Pot
Sebagai *sistem backend*, saya ingin berlangganan topik telemetri dengan pola wildcard agar data dari satu maupun banyak perangkat pot tertangkap oleh satu consumer.

- **REQ-202-01 (Ubiquitous)**: THE SYSTEM SHALL berlangganan topik `smartgrow/pot/+/telemetry` dengan jaminan kualitas pengiriman **QoS 1** (*at least once*).
- **REQ-202-02 (Event-driven)**: WHEN koneksi MQTT pulih setelah terputus (*on connect handler*), THE SYSTEM SHALL mendaftarkan ulang (*re-subscribe*) seluruh topik langganan secara otomatis.

### US-203: Parsing & Deserialisasi Payload Telemetri
Sebagai *sistem backend*, saya ingin mengubah payload byte JSON mentah dari ESP32 menjadi struct data Go yang terstruktur agar dapat diproses oleh layer persistensi dan gateway.

- **REQ-203-01 (Event-driven)**: WHEN payload MQTT baru masuk pada topik telemetri, THE SYSTEM SHALL mengekstrak string `device_id` dari payload JSON atau dari token topik URL, serta mem-parsing seluruh field sensor dan aktuator.
- **REQ-203-02 (Unwanted event)**: IF payload MQTT berisi format JSON rusak (*malformed syntax*), THEN THE SYSTEM SHALL mencatat log error pada level `WARN` beserta cuplikan raw payload dan mengabaikan pesan tanpa menghentikan worker.

### US-204: Validasi Integritas & Batas Ambang Sensor
Sebagai *sistem backend*, saya ingin memvalidasi rentang fisik nilai sensor agar data anomali atau *sensor failure* tidak mencemari database.

- **REQ-204-01 (State-driven)**: WHILE nilai sensor divalidasi, THE SYSTEM SHALL memverifikasi bahwa:
  - `sensors.light_lux` >= 0
  - `sensors.temperature_c` berada dalam rentang -20.0 s.d. 85.0 °C
  - `sensors.humidity_pct` berada dalam rentang 0.0 s.d. 100.0 %
  - `sensors.soil_moisture_pct` berada dalam rentang 0.0 s.d. 100.0 %
  - `actuators.grow_light_dimming_pct` bernilai bulat antara 0 s.d. 100
- **REQ-204-02 (Unwanted event)**: IF salah satu nilai metrik berada di luar rentang fisik yang ditentukan, THEN THE SYSTEM SHALL menandai pesan sebagai tidak valid (*validation error*) dan mencatat log peringatan.

---

## 2. Non-Functional Requirements (NFR)

1. **Latensi Pemrosesan Pesan (Message Parsing Latency)**:
   - Waktu pemrosesan dari penerimaan pesan MQTT di callback hingga selesai parsing dan validasi struct tidak boleh melebihi 5 milidetik per pesan.
2. **Ketahanan Beban (Throughput & Concurrency)**:
   - Modul consumer harus mampu menangani throughput minimal 500 pesan/detik secara konkuren tanpa *memory leak* atau *goroutine leak*.
3. **Graceful Shutdown**:
   - Saat backend menerima sinyal terminasi (`SIGINT`/`SIGTERM`), subscriber MQTT harus memutuskan koneksi (*disconnect*) ke broker secara sopan dengan batas waktu tunggu (*quiesce timeout*) maksimal 1 detik.

---

## 3. Rujukan Dokumen
- [Techstack Architecture - MQTT Payload Specification](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L72-L115)
- [Design Fase 0 - Environment Variables](../00-project-bootstrap-environment/design.md#2-kontrak-variabel-lingkungan-configuration-contract)
- [Design Fase 1 - Skema Tabel Readings](../01-database-schema-migration/design.md#22-tabel-readings)
