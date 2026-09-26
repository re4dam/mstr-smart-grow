# Daftar Tugas Implementasi (Tasks) — Fase 8: Frontend: Analytics & Historical View

Daftar checklist tugas implementasi granular untuk halaman analitik dan grafik historis. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Setup Library Grafik & Komponen Kontrol
- [ ] **TASK-801**: Pasang library grafik (misal `npm install recharts`) ke dependensi `frontend/package.json`.
- [ ] **TASK-802**: Buat komponen `TimeRangeSelector.tsx` dengan pilihan tombol rentang waktu (`1h`, `6h`, `24h`, `7d`, `30d`).
- [ ] **TASK-803**: Buat komponen `PeriodSelector.tsx` untuk filter ringkasan metrik analitik (`today`, `this_week`, `this_month`).

## 2. Implementasi Grafik Deret Waktu (Charts)
- [ ] **TASK-804**: Buat komponen `LightVsDimmingChart.tsx` dengan dua sumbu Y (kiri untuk Lux, kanan untuk persentase dimming 0-100%).
- [ ] **TASK-805**: Buat komponen `ClimateChart.tsx` yang memvisualisasikan suhu (°C) dan kelembapan udara (%) secara bersamaan.
- [ ] **TASK-806**: Buat komponen `SoilMoistureHistoryChart.tsx` yang menampilkan fluktuasi kelembapan media tanam lengkap dengan area pita ambang ideal (*reference band* 40%-70%).

## 3. Implementasi Kartu Metrik Analitik & Integrasi Halaman
- [ ] **TASK-807**: Buat komponen `EnergySavingsCard.tsx` yang menampilkan perbandingan bar baseline vs actual kWh dan badge persentase penghematan energi.
- [ ] **TASK-808**: Buat komponen `BiologicalScoreCard.tsx` dan `ActionableRecommendationsCard.tsx` untuk merender skor tanaman serta daftar tips perawatan.
- [ ] **TASK-809**: Satukan seluruh komponen ke dalam `frontend/app/routes/analytics.tsx` dan hubungkan dengan fungsi fetch data rentang waktu secara reaktif.
- [ ] **TASK-810**: Buat fungsi unduh data historis dasar (format CSV) saat tombol ekspor ditekan.

---

## Rujukan Dokumen
- [Requirements Fase 8](requirements.md)
- [Design Fase 8](design.md)
