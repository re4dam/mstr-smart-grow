# Daftar Tugas Implementasi (Tasks) — Fase 7: Frontend: Real-time Monitoring View

Daftar checklist tugas implementasi granular untuk halaman pemantauan langsung (*Live Monitoring View*). Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Komponen Kartu Indikator Metrik
- [ ] **TASK-701**: Buat komponen `LightIntensityCard.tsx` dengan visualisasi nilai lux, ikon matahari, dan badge tingkat kecukupan cahaya.
- [ ] **TASK-702**: Buat komponen `TemperatureCard.tsx` dan `HumidityCard.tsx` untuk menampilkan parameter mikroklimat sekitar pot.
- [ ] **TASK-703**: Buat komponen `SoilMoistureCard.tsx` dengan indikator level kelembapan tanah (0-100%) dan pewarnaan status dinamis (Ideal/Kering/Basah).
- [ ] **TASK-704**: Buat komponen `GrowLightCard.tsx` yang menampilkan persentase dimming PWM LED (0-100%), estimasi daya saat ini, dan mode kerja ("Adaptive").

## 2. Integrasi State Reaktif & Watchdog
- [ ] **TASK-705**: Susun komponen kartu ke dalam grid responsif pada file halaman `frontend/app/routes/monitoring.tsx`.
- [ ] **TASK-706**: Hubungkan state halaman dengan event `telemetry_update` dari context WebSocket sehingga metrik terbarui secara langsung tanpa refresh halaman.
- [ ] **TASK-707**: Buat custom hook `useStaleData.ts` yang memantau selisih waktu antara timestamp lokal dan timestamp paket telemetri terakhir (tampilkan peringatan jika jeda > 30 detik).

## 3. Optimasi Render & Aksesibilitas
- [ ] **TASK-708**: Terapkan `React.memo` pada masing-masing komponen kartu metrik untuk mencegah re-render yang tidak perlu saat metrik lain berubah.
- [ ] **TASK-709**: Tambahkan animasi transisi CSS halus (ease-in-out) pada progress bar dan angka pembacaan sensor.
- [ ] **TASK-710**: Lakukan pengujian langsung: jalankan backend, injeksi payload tiruan via MQTT, dan verifikasi nilai di browser terbarui seketika (< 50ms).

---

## Rujukan Dokumen
- [Requirements Fase 7](requirements.md)
- [Design Fase 7](design.md)
