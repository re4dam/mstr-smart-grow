# Spesifikasi Kebutuhan (Requirements) — Fase 8: Frontend: Analytics & Historical View

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk halaman analitik dan grafik historis (*Historical Analytics & Energy Insights View*), visualisasi tren deret waktu multi-sensor, tampilan penghematan energi listrik (*kWh savings*), rekomendasi biologis tanaman, serta opsi unduh data.

---

## 1. User Stories & Acceptance Criteria (EARS Format)

### US-801: Visualisasi Grafik Deret Waktu Historis (Time-Series Charts)
Sebagai *pengguna*, saya ingin melihat grafik riwayat intensitas cahaya, suhu, kelembapan, dan persentase peredupan lampu dalam kurun waktu tertentu agar saya dapat mengevaluasi tren kestabilan mikroklimat tanaman.

- **REQ-801-01 (Ubiquitous)**: THE SYSTEM SHALL menyediakan grafik visualisasi tren interaktif dengan sumbu waktu (X) dan nilai sensor (Y).
- **REQ-801-02 (Event-driven)**: WHEN pengguna memilih filter rentang waktu (`1h`, `6h`, `24h`, `7d`, `30d`), THE SYSTEM SHALL memanggil endpoint `GET /api/readings?range={range}` dan memperbarui kurva grafik.
- **REQ-801-03 (State-driven)**: WHILE kursor pengguna diarahkan ke titik data tertentu pada kurva (*hover tooltip*), THE SYSTEM SHALL menampilkan tanggal, jam, dan nilai spesifik pada koordinat tersebut.

### US-802: Panel Metrik Penghematan Energi (kWh Savings)
Sebagai *pengguna*, saya ingin melihat kalkulasi jumlah energi listrik yang berhasil dihemat oleh sistem *adaptive grow lighting* dibandingkan dengan skema lampu konvensional (non-dimming) agar saya dapat melihat efisiensi biaya listrik secara nyata.

- **REQ-802-01 (Ubiquitous)**: THE SYSTEM SHALL menampilkan ringkasan metrik energi dari endpoint `GET /api/analytics/summary` yang memuat:
  - Konsumsi energi dasar (*Baseline kWh*).
  - Konsumsi energi aktual (*Actual kWh*).
  - Energi yang dihemat (*kWh Savings*).
  - Persentase efisiensi energi (*Savings Percentage %*).
- **REQ-802-02 (Event-driven)**: WHEN pengguna mengubah filter periode analitik (`today`, `this_week`, `this_month`), THE SYSTEM SHALL menyegarkan metrik ringkasan energi sesuai periode tersebut.

### US-803: Rekomendasi Biologis & Skor Tanaman
Sebagai *pengguna*, saya ingin membaca skor kesehatan tanah, skor kecukupan cahaya, dan rekomendasi perawatan tanaman yang dihasilkan oleh backend agar saya dapat melakukan tindakan yang tepat (misalnya menyiram pot).

- **REQ-803-01 (Ubiquitous)**: THE SYSTEM SHALL merender skor kepatuhan biologis (skala 0 - 100) serta daftar kartu teks rekomendasi tindakan perawatan tanaman.
- **REQ-803-02 (State-driven)**: WHILE terdapat rekomendasi berstatus peringatan atau urgensi tinggi, THE SYSTEM SHALL menandai kartu rekomendasi dengan ikon peringatan yang mencolok.

### US-804: Ekspor Data Historis (Opsional)
Sebagai *pengguna*, saya ingin mengunduh rekaman data historis ke dalam berkas CSV atau JSON agar dapat dianalisis lebih lanjut menggunakan perangkat lunak eksternal.

- **REQ-804-01 (Optional feature)**: WHERE tombol *Export Data* ditekan oleh pengguna, THE SYSTEM SHALL menghasilkan dan mengunduh berkas data deret waktu yang sedang ditampilkan.

---

## 2. Non-Functional Requirements (NFR)

1. **Kelancaran Render Grafik (Chart Rendering Performance)**:
   - Grafik harus mampu merender hingga 1.000 titik data time-series secara interaktif tanpa *lag* pergerakan kursor (waktu render awal < 300ms).
2. **Keterbacaan Visual (Data Legibility)**:
   - Garis kurva antar metrik yang berbeda (mis. Lux vs Dimming %) harus menggunakan warna kontras yang berbeda dan dilengkapi legenda yang jelas.
3. **Resiliensi Data Kosong (Empty States)**:
   - Apabila perangkat pot belum memiliki data historis pada rentang waktu yang dipilih, halaman harus menampilkan komponen *Empty State* yang ramah pengguna alih-alih grafik kosong yang rusak.

---

## 3. Rujukan Dokumen
- [Design Fase 4 - Kontrak REST API](../04-backend-rest-api/design.md#2-spesifikasi-endpoint--kontrak-respon)
- [Design Fase 6 - App Shell & Context](../06-frontend-dashboard-skeleton/design.md#1-struktur-rute-aplikasi-react-router)
- [Glosarium Istilah - kWh Savings](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/glossary.md#L32)
