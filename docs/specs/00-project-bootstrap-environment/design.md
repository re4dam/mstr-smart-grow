# Desain Teknis (Technical Design) — Fase 0: Project Bootstrap & Environment

Dokumen ini memuat keputusan arsitektur, struktur repositori, spesifikasi kontrak konfigurasi, topologi kontainer Docker Compose lokal, serta konfigurasi linter.

---

## 1. Struktur Repositori Monorepo

Struktur direktori dirancang untuk memisahkan backend Go, frontend React Router, dan perkakas orkestrasi lokal:

```text
mstr-smart-grow/
├── .gitignore
├── docker-compose.yml
├── .env.example
├── backend/
│   ├── cmd/
│   │   └── server/
│   │       └── main.go
│   ├── internal/
│   │   └── config/
│   │       └── config.go
│   ├── .env.example
│   ├── .golangci.yml
│   ├── go.mod
│   └── go.sum
├── frontend/
│   ├── app/
│   ├── public/
│   ├── .env.example
│   ├── .eslintrc.json
│   ├── .prettierrc
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
└── docs/
    └── specs/
```

---

## 2. Kontrak Variabel Lingkungan (*Configuration Contract*)

### 2.1 Backend (`backend/.env`)

| Variabel | Tipe Data | Nilai Default / Contoh | Wajib? | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| `APP_ENV` | `string` | `development` | Ya | `development`, `staging`, `production` |
| `PORT` | `integer` | `8080` | Ya | Port HTTP & WebSocket server |
| `MQTT_BROKER_URL` | `string` | `tcp://localhost:1883` | Ya | Format: `tcp://host:1883` atau `ssl://host:8883` |
| `MQTT_CLIENT_ID` | `string` | `smartgrow-backend-dev` | Ya | Client ID unik untuk MQTT |
| `MQTT_USERNAME` | `string` | *(kosong jika lokal)* | Tidak | Username autentikasi Mosquitto (password_file) |
| `MQTT_PASSWORD` | `string` | *(kosong jika lokal)* | Tidak | Password autentikasi Mosquitto |
| `MQTT_TOPIC` | `string` | `smartgrow/+/telemetry` | Ya | Pola topik langganan MQTT wildcard |
| `MARIADB_HOST` | `string` | `localhost` | Ya | Host server MariaDB |
| `MARIADB_PORT` | `integer` | `3306` | Ya | Port server MariaDB |
| `MARIADB_USER` | `string` | `smartgrow_user` | Ya | Username database |
| `MARIADB_PASSWORD` | `string` | `smartgrow_pass` | Ya | Password database |
| `MARIADB_DATABASE` | `string` | `smartgrow_db` | Ya | Nama skema database |

### 2.2 Frontend (`frontend/.env`)

| Variabel | Tipe Data | Nilai Default / Contoh | Wajib? | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| `VITE_API_BASE_URL` | `string` | `http://localhost:8080/api` | Ya | Base URL untuk REST API backend |
| `VITE_WS_URL` | `string` | `ws://localhost:8080/ws/live` | Ya | WebSocket endpoint untuk telemetri live |

---

## 3. Topologi Lingkungan Lokal (Docker Compose)

Untuk pengujian lokal terpadu, `docker-compose.yml` menyediakan MariaDB dan broker Eclipse Mosquitto lokal:

```mermaid
flowchart LR
    subgraph Host ["Host Machine"]
        BackendApp["Go Backend Service\n(:8080)"]
        FrontendApp["React Router Client\n(:5173)"]
    end

    subgraph DockerBridge ["Docker Bridge Network (smartgrow-net)"]
        MariaDBCont["MariaDB 11.x Container\nInternal: 3306\nExposed: 3306"]
        MQTTCont["MQTT Broker (Eclipse Mosquitto)\nInternal: 1883\nExposed: 1883"]
        VolumeDB[("Volume:\nmariadb_data")]
    end

    MariaDBCont --- VolumeDB
    BackendApp -->|SQL Queries| MariaDBCont
    BackendApp -->|MQTT Connect / Sub| MQTTCont
    FrontendApp -->|REST / WS| BackendApp
```

### Konfigurasi Service Docker Compose:
- **`mariadb`**: Image `mariadb:11.4`, *healthcheck* menggunakan `mariadb-admin ping -h localhost`, *persistent volume* `mariadb_data`.
- **`mqtt-broker`**: Image resmi `eclipse-mosquitto:2` pada port 1883 (MQTT standard) dan opsional 9001 (WebSocket MQTT), memuat berkas konfigurasi lokal `docker/mosquitto/mosquitto.conf` (`listener 1883`, `allow_anonymous true` untuk dev lokal).

---

## 4. Konfigurasi Linter & Tooling

1. **Backend Go**:
   - Pustaka config: `github.com/caarlos0/env/v11` atau standard library parser dengan fallback.
   - Pustaka linter: `.golangci.yml` mengaktifkan `errcheck`, `gosimple`, `govet`, `ineffassign`, `staticcheck`, dan `unused`.
2. **Frontend React Router**:
   - Pustaka bundler: Vite 5+ / React Router v7.
   - ESLint: `@typescript-eslint/recommended`, `eslint-plugin-react-hooks`.
   - Prettier: `semi: true`, `singleQuote: true`, `trailingComma: "es5"`.

---

## 5. Trade-Offs & Keputusan Desain

1. **Monorepo vs Multi-repo**:
   - *Keputusan*: Monorepo tunggal dipilih karena mempermudah koordinasi perubahan kontrak antara API backend dan dashboard frontend serta menyederhanakan konfigurasi CI/CD.
2. **Self-Hosted Mosquitto vs Cloud-Managed Broker**:
   - *Keputusan*: Mengadopsi Eclipse Mosquitto self-hosted berbasis container Docker resmi (`eclipse-mosquitto:2`). Hal ini menekan biaya operasional, menyederhanakan pengujian lokal (footprint memori ~10MB), dan memberikan kontrol penuh atas konfigurasi broker via `mosquitto.conf`.
3. **TODO Tim**:
   - `TODO: Tentukan mekanisme autentikasi Mosquitto untuk production (apakah menggunakan password_file via mosquitto_passwd atau plugin auth eksternal), serta evaluasi apakah perlu mengekspos port 9001 untuk WebSocket MQTT langsung ke edge/client.`

---

## 6. Rujukan Dokumen
- [Requirements Fase 0](requirements.md)
- [Techstack Architecture](file:///home/readam/Zaki-Adam/Development/mstr-smart-grow/docs/architecture/Techstack.md)
