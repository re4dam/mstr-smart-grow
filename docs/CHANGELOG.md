# Changelog

Semua perubahan penting pada proyek **Smart Grow Pot** akan dicatat dalam dokumen ini.

Format dokumen ini mengacu pada [Keep a Changelog](https://keepachangelog.com/id/1.1.0/),
dan proyek ini mematuhi prinsip [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- Setup fondasi service backend Go:
  - Integrasi MQTT consumer menggunakan pustaka `paho.mqtt.golang` untuk berlangganan topik HiveMQ.
  - Setup koneksi dan skema penyimpanan dasar time-series pada MariaDB.
  - Setup WebSocket Gateway untuk *real-time broadcasting* data telemetri dari MQTT ke browser klien.
  - Implementasi REST API awal (`GET /api/readings/latest`, `GET /api/readings`, `GET /api/analytics/summary`, dan `GET /api/health`).
  - Mesin kalkulasi analitik dasar untuk estimasi penghematan energi listrik (*kWh savings*).
- Setup frontend dashboard berbasis React Router:
  - Kerangka aplikasi (*skeleton*) dan struktur routing utama.
  - Komponen koneksi WebSocket untuk mendengarkan live feed telemetri pot.
  - Komponen visualisasi grafik historis menggunakan data dari REST API.
- Dokumentasi teknis proyek di folder `docs/`:
  - `README.md` (panduan setup dan struktur folder repo).
  - `architecture/Techstack.md` (matriks teknologi, diagram alur data Mermaid, spesifikasi MQTT dan API).
  - `overview.md` (narasi operasional end-to-end dan pembagian tanggung jawab tim).
  - `glossary.md` (daftar istilah domain dan teknis).
  - `change-requests-log.md` (catatan pelacakan perubahan arsitektur).
  - `specs/` (spesifikasi pengembangan bertahap Fase 0 s.d. 9 dengan requirements EARS, desain teknis, dan tasks granular).

### Changed
- *(Belum ada perubahan)*

### Fixed
- *(Belum ada perbaikan bug)*

### Removed
- *(Belum ada komponen yang dihapus)*

---

<!-- Rilis versi berikutnya akan ditambahkan di sini saat tagging versi dilakukan (misal: [0.1.0] - YYYY-MM-DD) -->
