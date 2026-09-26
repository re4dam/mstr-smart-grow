# Daftar Tugas Implementasi (Tasks) — Fase 6: Frontend: Dashboard Skeleton (React Router)

Daftar checklist tugas implementasi granular untuk fondasi kerangka frontend dashboard. Setiap tugas dirancang untuk diselesaikan dalam 1 commit / PR.

---

## 1. Setup Layout & Routing
- [ ] **TASK-601**: Konfigurasi struktur routing pada `frontend/app/routes/` mencakup root redirect, rute `/monitoring`, dan rute `/analytics`.
- [ ] **TASK-602**: Buat komponen `Navbar.tsx` dan `Header.tsx` untuk navigasi antar halaman dan pemilih pot.
- [ ] **TASK-603**: Implementasikan komponen `ConnectionBadge.tsx` untuk menampilkan status visual koneksi server (hijau untuk online, kuning untuk reconnecting, abu-abu/merah untuk disconnected).
- [ ] **TASK-604**: Buat komponen `ErrorBoundary.tsx` untuk menangani crash render atau kegagalan pemanggilan data secara anggun.

## 2. Layer Data Fetching (REST API Client)
- [ ] **TASK-605**: Buat modul service `frontend/app/services/api.ts` yang mengabstraksi pemanggilan `fetch` ke backend API dengan penanganan timeout dan standardisasi error.
- [ ] **TASK-606**: Implementasikan React Router loader function pada rute `/monitoring` untuk mengambil data `GET /api/readings/latest`.
- [ ] **TASK-607**: Implementasikan React Router loader function pada rute `/analytics` untuk memuat data awal `GET /api/analytics/summary`.

## 3. Layer WebSocket Client & State
- [ ] **TASK-608**: Buat `WebSocketContext.tsx` dan custom hook `useWebSocket.ts` yang mengelola koneksi browser ke `VITE_WS_URL`.
- [ ] **TASK-609**: Implementasikan logika auto-reconnect dengan backoff interval saat koneksi WebSocket terputus.
- [ ] **TASK-610**: Bungkus komponen `RootLayout` dengan `WebSocketProvider` dan verifikasi koneksi berhasil tersambung ke backend Go yang aktif.

---

## Rujukan Dokumen
- [Requirements Fase 6](requirements.md)
- [Design Fase 6](design.md)
