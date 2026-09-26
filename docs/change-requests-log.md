# Log Pengajuan Perubahan (Change Requests Log) — Smart Grow Pot

Dokumen ini mencatat seluruh usulan perubahan signifikan (*change requests*) yang memengaruhi arsitektur sistem, pemilihan pustaka/teknologi, skema data telemetri, kontrak API, atau perangkat keras proyek Smart Grow Pot.

---

## Tabel Pelacakan Change Request

| ID | Tanggal | Diajukan oleh | Deskripsi Perubahan | Alasan | Status | Terkait Dokumen / Modul |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CR-001** | 2026-09-24 | Backend Team | Migrasi penyimpanan data time-series dari MariaDB ke database khusus time-series (seperti TimescaleDB atau InfluxDB) | MariaDB dipilih sebagai solusi sementara. Jika frekuensi ingest data dari multi-pot meningkat pesat, database time-series menawarkan kompresi penyimpanan lebih tinggi dan performa agregasi rentang waktu (downsampling) yang jauh lebih cepat. | `Proposed` | [Techstack.md](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md), [overview.md](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/overview.md) |
| **CR-002** | *YYYY-MM-DD* | *(Role Tim)* | *(Template baris usulan baru: deskripsikan perubahan yang diajukan)* | *(Alasan teknis atau bisnis di balik pengajuan)* | `Proposed` / `Approved` / `Rejected` / `Implemented` | `docs/...` |

---

## Panduan Pengajuan Perubahan

1. **ID**: Gunakan format berurutan `CR-XXX` (misal `CR-003`).
2. **Status**:
   - `Proposed`: Usulan baru diajukan dan sedang menunggu evaluasi teknis tim.
   - `Approved`: Disetujui tim dan siap dijadwalkan untuk implementasi.
   - `Rejected`: Ditolak setelah tinjauan dengan catatan alasan penolakan.
   - `Implemented`: Telah selesai diimplementasikan dan diverifikasi di kode sumber.
3. **Dokumentasi Terkait**: Cantumkan tautan ke dokumen di `docs/` atau modul kode yang terdampak oleh perubahan tersebut.
