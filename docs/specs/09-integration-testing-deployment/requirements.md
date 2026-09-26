# Spesifikasi Kebutuhan (Requirements) — Fase 9: Integration Testing & Deployment

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk pengujian integrasi menyeluruh (*End-to-End Testing*) dengan simulasi perangkat keras, strategi kontainerisasi produksi (*Docker multi-stage build*), manajemen rilis, serta pemantauan log operasional (*production monitoring & structured logging*).

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-901: Simulasi End-to-End Perangkat ESP32
Sebagai *QA / Pengembang*, saya ingin menjalankan simulator perangkat ESP32 yang memublikasikan payload telemetri realistik ke broker MQTT agar seluruh alur sistem (Edge → MQTT → Go Backend → DB → Dashboard) dapat diuji secara otomatis tanpa perangkat fisik.

- **REQ-901-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan skrip simulator perangkat (berbasis Python atau Go) yang mampu menyimulasikan profil perubahan intensitas cahaya, suhu, dan kelembapan secara kontinu.
- **REQ-901-02 (Event-driven)**: WHEN skrip simulator memublikasikan payload ke broker HiveMQ, THE SYSTEM SHALL memverifikasi bahwa:
  1. Data tersimpan di tabel `readings` MariaDB.
  2. Data disiarkan via WebSocket ke klien yang terhubung dalam waktu < 200ms.
  3. Endpoint REST `GET /api/readings/latest` langsung mencerminkan data terbaru.

### US-902: Kontainerisasi Produksi Backend (Multi-Stage Build)
Sebagai *DevOps Engineer*, saya ingin membangun image Docker backend Go yang terisolasi dan berukuran minimal agar proses *deployment* cepat dan aman dari celah keamanan sistem operasi dasar.

- **REQ-902-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan `Dockerfile` multi-stage untuk backend Go dengan stage akhir berbasis image *distroless* atau *alpine* berbobot ringan.
- **REQ-902-02 (State-driven)**: WHILE image Docker backend di-build, THE SYSTEM SHALL mengkompilasi binary Go secara statis (`CGO_ENABLED=0`) dengan ukuran image akhir < 35 MB.

### US-903: Build & Hosting Frontend Produksi
Sebagai *DevOps Engineer*, saya ingin mem-bundle aplikasi React Router ke dalam artefak statis produksi yang teroptimasi agar dapat disajikan secara efisien oleh web server (Nginx/Caddy) atau CDN.

- **REQ-903-01 (Event-driven)**: WHEN perintah `npm run build` dijalankan, THE SYSTEM SHALL menghasilkan file HTML, CSS, dan JS yang ter-minifikasi tanpa artefak pengembangan (*source maps non-exposed* pada produksi).
- **REQ-903-02 (Ubiquitous)**: THE SYSTEM SHALL menyediakan konfigurasi web server produksi (Nginx/Caddy) yang menyajikan file statis dan mendukung *fallback routing* ke `index.html` untuk SPA / client-side routing.

### US-904: Pemantauan & Pencatatan Terstruktur (Structured Logging)
Sebagai *Operator Sistem*, saya ingin backend memancarkan log terstruktur dalam format JSON agar mudah dikumpulkan dan dianalisis oleh agregator log (misal Loki, Datadog, atau CloudWatch).

- **REQ-904-01 (Ubiquitous)**: THE SYSTEM SHALL menggunakan modul `log/slog` bawaan Go untuk seluruh pencatatan log dengan format JSON pada level produksi (`INFO`, `WARN`, `ERROR`).
- **REQ-904-02 (State-driven)**: WHILE terjadi error tak terduga (*panic* atau kegagalan koneksi berulang), THE SYSTEM SHALL mencatat stack trace dan metadata konteks (`device_id`, timestamp, error message).

---

## 2. Non-Functional Requirements (NFR)

1. **Keamanan Kontainer (Container Security)**:
   - Kontainer produksi backend dan frontend harus berjalan di bawah pengguna non-root (*non-root user* UID 10001).
2. **Ketersediaan & Pemulihan (High Availability & Health Probes)**:
   - Kontainer backend harus menyertakan konfigurasi liveness dan readiness probe yang memanggil endpoint `GET /api/health`.
3. **Waktu Rilis CI/CD (Pipeline Build Time)**:
   - Seluruh tahapan pipeline CI/CD (test, lint, build docker) tidak boleh memakan waktu lebih dari 5 menit.

---

## 3. Rujukan Dokumen
- [Techstack Architecture - Deployment Matrix](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L17)
- [Design Fase 4 - Health Check API](../04-backend-rest-api/design.md#22-endpoint-detail)
- [Design Fase 5 - WebSocket Broadcast](../05-backend-websocket-gateway/design.md#3-sequence-diagram-broadcast--heartbeat)
