# 📑 Panduan Instalasi & Konfigurasi Grafana + Loki + Promtail (Docker Compose)

Dokumentasi ini menjelaskan proses **instalasi, konfigurasi, dan validasi Grafana, Loki, dan Promtail** menggunakan **Docker Compose** pada **Ubuntu Server 24.04 LTS** untuk lingkungan **production / data center / enterprise**.

---

# 🖥️ BAGIAN 1 — HOST (Grafana + Loki)

Host berfungsi sebagai **server monitoring** sekaligus **log aggregation server** yang menerima log dari seluruh agent menggunakan Promtail, menyimpannya di Loki, kemudian divisualisasikan melalui Grafana.

---

## 🧰 Prasyarat

- OS : **Ubuntu Server 24.04 LTS**
- Docker Engine dan Docker Compose Plugin **sudah terpasang**
- Hak akses `sudo`
- Koneksi jaringan antara Host dan Agent
- Port yang digunakan:
  - **3000/TCP** (Grafana)
  - **3100/TCP** (Loki)

---

## 🏗️ Arsitektur

```text
                 +----------------------+
                 |      GRAFANA         |
                 |      Port 3000       |
                 +----------+-----------+
                            |
                            |
                 +----------v-----------+
                 |        LOKI          |
                 |      Port 3100       |
                 +----------+-----------+
                            ^
                            |
        -----------------------------------------------
        |                   |                         |
+----------------+  +----------------+      +----------------+
| Promtail Node1 |  | Promtail Node2 |  ... | Promtail NodeN |
+----------------+  +----------------+      +----------------+
```

---

# 🌐 STEP 1 : Membuat Direktori

```bash
mkdir -p /opt/monitoring/loki
mkdir -p /opt/monitoring/grafana
cd /opt/monitoring
```

---

# 🌐 STEP 2 : Konfigurasi Loki

Buat file konfigurasi.

```bash
nano loki/loki.yaml
```

Isi file:

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9095


common:
  instance_addr: 127.0.0.1
  path_prefix: /loki

  replication_factor: 1

  ring:
    kvstore:
      store: inmemory

  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules


schema_config:
  configs:
    - from: 2024-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h


storage_config:
  tsdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache

  filesystem:
    directory: /loki/chunks


limits_config:
  allow_structured_metadata: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h


compactor:
  working_directory: /loki/compactor
  retention_enabled: true


analytics:
  reporting_enabled: false
```

---

# 🌐 STEP 3 : Membuat Docker Compose

Buat file:

```bash
nano docker-compose.yml
```

Isi file:

```yaml
version: "3.9"

services:

  loki:
    image: grafana/loki:latest
    container_name: loki
    restart: unless-stopped

    command:
      - '-config.file=/etc/loki/loki.yml'

    ports:
      - "3100:3100"

    volumes:
      - ./loki/loki.yml:/etc/loki/loki.yml:ro
      - loki_data:/loki

    networks:
      - monitoring

    healthcheck:
      test:
        - CMD
        - wget
        - --spider
        - -q
        - http://localhost:3100/ready
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 20s

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped

    ports:
      - "3000:3000"

    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: CHANGE_THIS_STRONG_PASSWORD
      GF_USERS_ALLOW_SIGN_UP: "false"

    volumes:
      - grafana_data:/var/lib/grafana

    depends_on:
      loki:
        condition: service_healthy

    networks:
      - monitoring

    healthcheck:
      test:
        - CMD
        - wget
        - --spider
        - -q
        - http://localhost:3000/api/health
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s

volumes:
  loki_data:
  grafana_data:

networks:
  monitoring:
    driver: bridge
```

---

# 🌐 STEP 4 : Menjalankan Container

```bash
docker compose up -d
```

Verifikasi container.

```bash
docker ps
```

---

# 🌐 STEP 5 : Konfigurasi Grafana

Akses Grafana melalui browser.

```
http://IP_SERVER:3000
```

Login menggunakan akun bawaan.

```
Username : admin
Password : admin
```

Tambahkan **Data Source**.

```
Connections
→ Add Data Source
→ Loki
```

Masukkan URL berikut.

```
http://loki:3100
```

Kemudian klik **Save & Test**.

---

# 📁 STEP 6 : Struktur Direktori

```text
grafana-loki/
├── docker-compose.yml
├── grafana/
└── loki/
    └── config.yaml
```

---

# ✅ STEP 7 : Validasi

Melihat status container.

```bash
docker ps
```

Memastikan Loki siap menerima koneksi.

```bash
curl http://localhost:3100/ready
```

Output yang diharapkan.

```
ready
```

---

# 🔐 STEP 8 : Keamanan

- Batasi akses port **3000** dan **3100** menggunakan firewall.
- Gunakan HTTPS melalui Reverse Proxy (Nginx atau Traefik).
- Ganti username dan password bawaan Grafana.
- Gunakan Docker Volume untuk penyimpanan persisten.
- Lakukan backup volume Grafana dan Loki secara berkala.

---

# 📊 STEP 9 : Monitoring & Logging

Melihat status container.

```bash
docker ps
```

Log Grafana.

```bash
docker logs -f grafana
```

Log Loki.

```bash
docker logs -f loki
```

---

---

# 💻 BAGIAN 2 — AGENT (Promtail)

Promtail bertugas membaca log pada server kemudian mengirimkannya ke Loki.

---

## 🧰 Prasyarat

- Ubuntu Server 24.04 LTS
- Docker Engine dan Docker Compose Plugin **sudah terpasang**
- Hak akses `sudo`
- Dapat mengakses Host Loki
- Port **3100/TCP** dapat dijangkau

---

# 🌐 STEP 1 : Membuat Direktori

```bash
mkdir -p /opt/promtail
cd /opt/promtail
```

---

# 🌐 STEP 2 : Konfigurasi Promtail

Buat file konfigurasi.

```bash
nano promtail.yaml
```

Isi file.

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://IP_SERVER:3100/loki/api/v1/push

scrape_configs:

- job_name: system

  static_configs:

  - targets:
      - localhost

    labels:
      job: syslog
      host: node01
      __path__: /var/log/*.log
```

> Ganti **IP_SERVER** dengan alamat IP Host yang menjalankan Loki.

---

# 🌐 STEP 3 : Membuat Docker Compose

Buat file.

```bash
nano docker-compose.yml
```

Isi file.

```yaml
version: "3.9"

services:

  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    restart: unless-stopped

    command:
      - -config.file=/etc/promtail/promtail.yml

    volumes:
      - ./config.yaml:/etc/promtail/promtail.yml
      - /var/log:/var/log:ro
      - /etc/machine-id:/etc/machine-id:ro
```

---

# 🌐 STEP 4 : Menjalankan Container

```bash
docker compose up -d
```

---

# 🌐 STEP 5 : Validasi

Melihat status container.

```bash
docker ps
```

Melihat log Promtail.

```bash
docker logs -f promtail
```

Menguji koneksi ke Loki.

```bash
curl http://IP_SERVER:3100/ready
```

Output yang diharapkan.

```
ready
```

---

# 🌐 STEP 6 : Verifikasi di Grafana

Masuk ke menu.

```
Explore
```

Pilih **Loki** kemudian jalankan query berikut.

```logql
{job="syslog"}
```

Apabila log berhasil ditampilkan, maka Promtail telah berhasil mengirimkan log ke Loki.

---

# 🔐 STEP 7 : Keamanan

- Batasi akses Promtail hanya ke Host Loki.
- Gunakan jaringan internal (*private network*) apabila memungkinkan.
- Terapkan firewall pada Host dan Agent.
- Jalankan container menggunakan user non-root apabila diperlukan.
- Lakukan backup konfigurasi Promtail.

---

# 📊 STEP 8 : Monitoring & Logging

Melihat status container.

```bash
docker ps
```

Melihat log Promtail.

```bash
docker logs -f promtail
```

---

# 🧪 Troubleshooting

| Masalah | Penyebab | Solusi |
|----------|----------|---------|
| Grafana tidak dapat terhubung ke Loki | URL Data Source salah | Pastikan URL menggunakan `http://loki:3100` |
| Loki tidak berjalan | Konfigurasi salah | Periksa `config.yaml` dan jalankan `docker logs loki` |
| Promtail gagal mengirim log | IP atau port Loki salah | Periksa parameter `clients.url` |
| Log tidak muncul di Grafana | Path log tidak sesuai | Periksa nilai `__path__` pada `config.yaml` |
| Container berhenti (*Exited*) | Kesalahan konfigurasi | Jalankan `docker logs <container>` untuk melihat penyebab |

---

# 📌 Catatan Penting

> ⚠️ Seluruh perubahan konfigurasi disarankan diuji terlebih dahulu pada lingkungan **staging** sebelum diterapkan ke lingkungan **production**. Setelah implementasi selesai, lakukan backup terhadap file konfigurasi dan Docker Volume secara berkala untuk menjaga ketersediaan data.

---

# 📎 Referensi

- Grafana Documentation
- Loki Documentation
- Promtail Documentation
- Grafana Labs Best Practices
- CIS Ubuntu Benchmark
