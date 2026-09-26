# Spesifikasi Kebutuhan (Requirements) — Fase 7: Frontend: Real-time Monitoring View

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk halaman pemantauan langsung (*Live Monitoring View*), mencakup visualisasi instan metrik intensitas cahaya, suhu, kelembapan udara, kelembapan tanah, status *duty cycle* peredupan LED (*dimming percentage*), pembaruan state reaktif via WebSocket, serta indikasi keaktifan data.

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-701: Visualisasi Metrik Lingkungan & Tanah
Sebagai *pengguna*, saya ingin melihat pembacaan terkini dari sensor cahaya, suhu, kelembapan udara, dan kelembapan tanah dalam bentuk kartu indikator (*cards / gauges*) yang menarik dan informatif agar saya dapat mengetahui kondisi lingkungan pot secara seketika.

- **REQ-701-01 (Ubiquitous)**: THE SYSTEM SHALL menampilkan kartu metrik terpisah untuk:
  - Intensitas Cahaya (satuan lux, nilai dari sensor BH1750).
  - Suhu Udara (satuan °C, sensor DHT11/22).
  - Kelembapan Udara (satuan %, sensor DHT11/22).
  - Kelembapan Tanah (satuan %, Capacitive Soil Sensor) disertai label status biologis (`"Ideal"`, `"Kering"`, `"Terlalu Basah"`).
- **REQ-701-02 (State-driven)**: WHILE nilai kelembapan tanah berada di luar batas ideal (misal < 30% atau > 80%), THE SYSTEM SHALL menampilkan indikator visual peringatan warna (kuning/merah) pada kartu tanah.

### US-702: Status Aktuator Lampu & Peredupan Adaptif
Sebagai *pengguna*, saya ingin memantau persentase peredupan (*PWM dimming %*) lampu tumbuh dan mode operasinya (*Adaptive / Manual*) agar saya mengetahui seberapa besar kontribusi penghematan cahaya buatan saat ini.

- **REQ-702-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan kartu pemantau lampu LED yang menampilkan persentase *dimming* (0% - 100%), indikator visual tingkat kecerahan, dan badge mode pencahayaan (`"Adaptive"`).
- **REQ-702-02 (Event-driven)**: WHEN nilai persentase dimming berubah, THE SYSTEM SHALL memperbarui indikator tingkat kecerahan dengan transisi visual yang halus (*smooth transition*).

### US-703: Pembaruan State Real-Time Tanpa Polling (Seamless In-Memory Update)
Sebagai *pengguna*, saya ingin nilai pada kartu indikator terbarui secara instan saat ada pesan telemetri baru dari WebSocket tanpa memicu pemuatan ulang halaman (*no screen flash*) dan tanpa meminta ulang data ke REST API.

- **REQ-703-01 (Event-driven)**: WHEN pesan WebSocket bertipe `telemetry_update` diterima untuk `device_id` yang sedang aktif, THE SYSTEM SHALL memperbarui state lokal komponen dalam waktu kurang dari 50ms.
- **REQ-703-02 (Unwanted event)**: IF pesan telemetri yang diterima memiliki `device_id` yang berbeda dari pot yang sedang dipilih, THEN THE SYSTEM SHALL mengabaikan event tersebut tanpa memperbarui tampilan aktif.

### US-704: Indikator Kebaruan Data & Peringatan Offline (Stale Data Detection)
Sebagai *pengguna*, saya ingin mengetahui jika pot tanaman berhenti mengirimkan data telemetri agar saya menyadari adanya masalah koneksi fisik atau catu daya pada perangkat.

- **REQ-704-01 (State-driven)**: WHILE tidak ada payload baru yang diterima dari pot selama lebih dari 30 detik, THE SYSTEM SHALL menampilkan badge peringatan *"Data Usang / Pot Offline"* pada header pemantauan.
- **REQ-704-02 (Event-driven)**: WHEN paket telemetri baru kembali diterima setelah jeda, THE SYSTEM SHALL menghapus badge peringatan dan kembali ke status *"Live"*.

---

## 2. Non-Functional Requirements (NFR)

1. **Efisiensi Rendering (Render Optimization)**:
   - Pembaruan metrik dari WebSocket tidak boleh memicu rendering ulang (*re-render*) komponen yang tidak terdampak (manfaatkan `React.memo` atau fine-grained state hooks).
2. **Kelancaran Visual (60 FPS Animation)**:
   - Transisi pergerakan jarum gauge atau progress bar persentase harus beroperasi pada 60 FPS menggunakan CSS transitions / transforms tanpa memicu layout reflow berat.
3. **Aksesibilitas (A11y)**:
   - Seluruh kartu metrik harus menyertakan atribut ARIA (seperti `aria-live="polite"` dan `role="status"`) agar dapat dibaca oleh pembaca layar.

---

## 3. Rujukan Dokumen
- [Techstack Architecture - WebSocket Specification](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md#L208-L238)
- [Design Fase 5 - WebSocket Gateway](../05-backend-websocket-gateway/design.md#2-kontrak-frame-pesan-websocket)
- [Design Fase 6 - App Shell & Context](../06-frontend-dashboard-skeleton/design.md#3-desain-klien-websocket--mesin-status-koneksi)
