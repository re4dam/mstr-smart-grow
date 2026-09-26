# Gambaran Umum Sistem (System Overview) — Smart Grow Pot

Dokumen ini menjelaskan alur operasional *end-to-end* dari sistem Smart Grow Pot, arsitektur teknis data yang mengadopsi **Pola 1 (Backend sebagai MQTT Consumer + API + WebSocket Gateway)**, serta pembagian tanggung jawab lintas layer rekayasa (*engineering responsibilities*).

---

## 1. Alur Operasional Sistem End-to-End

Sistem Smart Grow Pot beroperasi secara siklis dan terintegrasi dari sensor di pot fisik hingga tampilan di layar pengguna:

1. **Penginderaan Fisik (*Sensing Layer*)**:
   - Mikrokontroler **ESP32** secara berkala (mis. setiap 3–5 detik) membaca data lingkungan mikro dari tiga modul sensor:
     - **BH1750**: mengukur intensitas cahaya sekitar dalam satuan lux.
     - **DHT11 / DHT22**: membaca suhu ruangan (°C) dan kelembapan relatif udara (% RH).
     - **Capacitive Soil Moisture Sensor**: mengukur kelembapan media tanam secara non-korosif.

2. **Eksekusi Logika Aturan Lokal (*Rule-Based Firmware Logic*)**:
   - Di dalam memori ESP32, firmware menjalankan dua modul evaluasi utama:
     - **Adaptive Lighting Logic**: Membandingkan pembacaan lux alami dengan kebutuhan target tanaman. Apabila ruangan terang, persentase *dimming* lampu LED grow light diturunkan atau dimatikan via modulasi sinyal PWM (MOSFET). Sebaliknya, saat cahaya meredup, output lampu dinaikkan secara proporsional.
     - **Biological Threshold Evaluation**: Membandingkan tingkat kelembapan tanah dan suhu terhadap batas ambang toleransi tanaman (mis. status "Ideal", "Mulai Kering", atau "Perlu Air").
   - Status terkini juga diperbarui secara langsung pada **display lokal LCD/OLED** yang terpasang pada fisik pot.

3. **Transmisi Telemetri (*MQTT Publish*)**:
   - ESP32 menyusun pembacaan sensor dan status aktuator ke dalam objek serialisasi JSON terstandarisasi.
   - Melalui modul Wi-Fi bawaan, ESP32 memublikasikan paket data tersebut ke **HiveMQ MQTT Broker** pada topik terstruktur (`smartgrow/pot/{pot_id}/telemetry`) dengan jaminan QoS 1.

4. **Konsumsi Data & Persistensi (*Go Backend Ingestion & Persistence*)**:
   - Service **Go Backend** yang bertindak sebagai subscriber utama (menggunakan pustaka `paho.mqtt.golang`) mendengarkan (*subscribe*) pesan yang masuk pada HiveMQ broker.
   - Ketika payload telemetri tiba, *goroutine worker* mem-parsing JSON dan menuliskan (*write/insert*) rekaman data ke database **MariaDB** untuk keperluan penyimpanan jejak historis (*time-series log*).

5. **Streaming Real-Time & Penyajian API (*WebSocket Gateway & REST API*)**:
   - Secara simultan, Go Backend menyalurkan (*broadcast*) payload telemetri terkini melalui **WebSocket Gateway** ke seluruh browser pengguna yang sedang membuka dashboard.
   - Go Backend juga menyediakan **REST API** untuk melayani kueri data historis (mis. `GET /api/readings?range=24h`), pembacaan terakhir (`GET /api/readings/latest`), serta penghitungan kalkulasi analitik (`GET /api/analytics/summary`) seperti estimasi penghematan energi (kWh savings) dan skor kepatuhan biologis.

6. **Visualisasi Interaktif (*React Router Dashboard*)**:
   - Aplikasi frontend berbasis **React Router** menerima stream data WebSocket untuk memperbarui indikator *gauge*, kartu status, dan persentase *dimming* secara langsung tanpa perlu me-refresh halaman (*no polling*).
   - React Router memanggil REST API untuk merender grafik tren lingkungan (time-series chart) serta panel rekomendasi perawatan tanaman.

---

## 2. Diagram Arsitektur Tingkat Tinggi

Diagram berikut mengilustrasikan interaksi komponen berdasarkan Pola 1:

```mermaid
flowchart TB
    subgraph PhysicalDevice ["Physical Pot & Edge Layer"]
        Sensors["Sensors:\n- BH1750 (Lux)\n- DHT11/22 (Temp & Hum)\n- Capacitive Soil Sensor"]
        ESP["ESP32 Microcontroller\n- Adaptive Lighting Logic\n- Biological Threshold Check"]
        Actuator["Actuators & Display:\n- PWM LED Grow Light\n- Local LCD/OLED Screen"]

        Sensors -->|Raw Readings| ESP
        ESP -->|PWM Dimming / Status Display| Actuator
    end

    subgraph MessagingBroker ["Message Broker Layer"]
        HiveMQ["HiveMQ MQTT Broker\nTopic: smartgrow/pot/+/telemetry"]
    end

    subgraph BackendSystem ["Go Backend Service"]
        MQTTSub["MQTT Consumer\n(paho.mqtt.golang)"]
        WSGateway["WebSocket Gateway\n(/ws/live)"]
        WorkerPool["Data Ingestion & Persistence Worker"]
        AnalyticsService["Analytics & Energy Calculator"]
        RESTRouter["REST API Router\n(/api/*)"]

        MQTTSub -->|Telemetry Stream| WSGateway
        MQTTSub -->|Telemetry Record| WorkerPool
        AnalyticsService -.->|Compute Metrics| RESTRouter
    end

    subgraph DatabaseLayer ["Data Persistence Layer"]
        MariaDB[("MariaDB\n(Time-Series Sensor Readings)")]
        WorkerPool -->|Insert Query| MariaDB
        RESTRouter -->|Select / Aggregate Query| MariaDB
    end

    subgraph WebClient ["Dashboard Layer (React Router)"]
        LiveView["Live Telemetry View\n(WebSocket Subscriber)"]
        HistoryView["Historical Analytics & Trends\n(REST API Consumer)"]
    end

    ESP -->|Wi-Fi / MQTT Publish (QoS 1)| HiveMQ
    HiveMQ -->|MQTT Subscribe| MQTTSub

    WSGateway -->|Live Feed Push| LiveView
    RESTRouter -->|JSON Responses| HistoryView
```

---

## 3. Pembagian Tanggung Jawab Tim (Team Responsibilities)

Untuk menjaga batas arsitektur (*separation of concerns*) yang jelas, tanggung jawab pengembangan dibagi menjadi tiga layer utama:

### 3.1 Layer Hardware & Firmware
- **Cakupan Tanggung Jawab**:
  - Perancangan skematik perkabelan sirkuit antara ESP32, sensor (BH1750, DHT11/22, Soil Sensor), aktuator MOSFET/LED, dan layar lokal LCD/OLED.
  - Penulisan firmware C++ di Arduino IDE yang efisien, non-blocking (menggunakan timer millis alih-alih `delay()`).
  - Implementasi *Adaptive Lighting Logic* untuk peredupan LED berbanding terbalik dengan cahaya alami.
  - Implementasi *Biological Threshold Evaluation* serta visualisasi status pada layar lokal pot.
  - Manajemen koneksi Wi-Fi (reconnect logic) dan publikasi data telemetri JSON via protokol MQTT ke broker HiveMQ.

### 3.2 Layer Backend Go & Database
- **Cakupan Tanggung Jawab**:
  - Pengelolaan koneksi subscriber ke broker HiveMQ menggunakan `paho.mqtt.golang` dengan penanganan reconnect otomatis.
  - Perancangan skema tabel MariaDB untuk penyimpanan data time-series sensor yang efisien.
  - Implementasi *WebSocket Gateway* untuk menyiarkan pesan MQTT yang masuk ke browser klien secara real-time.
  - Penyediaan endpoint REST API untuk kueri data historis dengan filter rentang waktu (*range filtering*).
  - Algoritma analitik sisi server: kalkulasi estimasi penghematan energi listrik (*kWh savings*) terhadap *baseline* non-dimming dan pembuatan rekomendasi biologis.
  - Pemantauan performa database MariaDB dan evaluasi kebutuhan migrasi ke engine time-series (TimescaleDB / InfluxDB).

### 3.3 Layer Frontend Dashboard (React Router)
- **Cakupan Tanggung Jawab**:
  - Pembangunan antarmuka pengguna responsif berbasis React Router.
  - Integrasi koneksi WebSocket client untuk pembaruan komponen indikator instan (lux, suhu, kelembapan tanah, status dimming %) tanpa latensi *polling*.
  - Visualisasi grafik tren historis (time-series chart) menggunakan data yang diambil dari REST API.
  - Panel analitik visual untuk menampilkan metrik efisiensi daya (kWh yang berhasil dihemat) serta notifikasi/peringatan ambang batas tanaman.
  - Penanganan state konektivitas (indikator status perangkat *Online/Offline*).
