# Glosarium Istilah — Smart Grow Pot

Dokumen ini berisi daftar istilah domain agrikultur/IoT dan istilah teknis perangkat lunak yang digunakan dalam proyek Smart Grow Pot. Istilah diurutkan secara alfabetis.

---

### A
- **Adaptive Lighting Logic**: Algoritma cerdas berbasis aturan pada firmware ESP32 yang mengatur persentase peredupan (*dimming*) lampu LED grow light secara dinamis dan berbanding terbalik dengan intensitas cahaya alami di sekitar pot.

### B
- **BH1750**: Modul sensor intensitas cahaya digital berbasis antarmuka I2C yang mengukur pencahayaan dalam satuan lux secara presisi.
- **Biological Threshold Evaluation**: Proses evaluasi logika pada firmware atau backend yang membandingkan nilai aktual sensor (kelembapan tanah, suhu, dan kelembapan udara) terhadap parameter ambang batas biologis ideal bagi varietas tanaman tertentu.

### C
- **Capacitive Soil Moisture Sensor**: Sensor kelembapan tanah berbasis perubahan kapasitansi dielektrik media tanam, memiliki ketahanan tinggi terhadap korosi dibandingkan sensor resistif konvensional.

### D
- **DHT11 / DHT22**: Modul sensor digital terpadu untuk mengukur suhu udara (dalam derajat Celsius) dan kelembapan relatif udara (relative humidity dalam persen).
- **Dimming Ratio (%)**: Rasio atau persentase siklus kerja (*duty cycle*) sinyal modulasi lebar pulsa (PWM) yang dikirimkan ke aktuator LED driver untuk mengontrol tingkat terang lampu tumbuh.

### E
- **ESP32**: Mikrokontroler berkemampuan Wi-Fi dan Bluetooth dengan prosesor dual-core yang berfungsi sebagai pemroses utama (*brain*) di sisi perangkat fisik (*edge*).

### G
- **Goroutine**: Utas eksekusi ringan (*lightweight thread*) yang dikelola oleh Go runtime, memungkinkan pemrosesan konkuren efisien tinggi untuk menangani konsumsi pesan MQTT dan koneksi WebSocket simultan.
- **Grow Light**: Sumber pencahayaan buatan (LED) dengan spektrum elektromagnetik yang dioptimalkan untuk memicu fotosintesis dan pertumbuhan vegetatif/generatif tanaman.

### H
- **HiveMQ**: Platform *message broker* MQTT berkinerja tinggi yang memfasilitasi pertukaran pesan secara terdistribusi dan aman antara ESP32 dan backend Go.

### K
- **kWh Savings (Penghematan kWh)**: Estimasi selisih antara konsumsi daya listrik lampu saat menyala 100% konstan (*baseline*) dengan konsumsi daya aktual berkat peredupan adaptif, dihitung dalam satuan kilowatt-hour.

### L
- **Lux (lx)**: Satuan turunan SI untuk iluminansi atau fluks cahaya per satuan luas, digunakan untuk mengukur intensitas cahaya alami yang mengenai sensor BH1750.

### M
- **MariaDB**: Sistem manajemen basis data relasional (RDBMS) berbasis SQL yang digunakan sebagai penyimpanan data telemetri historis pada tahap awal proyek.
- **MQTT Broker**: Node server perantara dalam arsitektur publish/subscribe MQTT yang menerima pesan dari pengirim (publisher) dan meneruskannya ke penerima yang berhak (subscriber).
- **MQTT Subscriber / Publisher**: Entitas klien MQTT; publisher bertugas memublikasikan data telemetri ke suatu topik (ESP32), sedangkan subscriber menerima data dari topik yang diminatinya (Go backend).
- **MQTT Topic**: String hierarkis bergaris miring (misalnya `smartgrow/pot/pot-01/telemetry`) yang digunakan broker MQTT untuk memfilter dan mengarahkan rute pesan kepada subscriber yang relevan.

### R
- **React Router**: Pustaka routing deklaratif untuk React yang mengelola navigasi halaman dan pemuatan data (*data loading*) pada dashboard web.
- **REST Endpoint**: Titik akhir URL spesifik pada server Go backend yang melayani permintaan HTTP (seperti `GET /api/readings`) menggunakan representasi data standar (JSON).

### T
- **Telemetry (Telemetri)**: Proses pengumpulan dan transmisi otomatis data pembacaan sensor dan status aktuator dari perangkat fisik jarak jauh ke sistem pusat untuk dipantau dan dianalisis.

### W
- **WebSocket Gateway**: Modul antarmuka jaringan pada backend Go yang mempertahankan koneksi persisten dua arah (*full-duplex*) dengan browser dashboard untuk menyiarkan pembaruan telemetri secara seketika (*real-time broadcast*).
