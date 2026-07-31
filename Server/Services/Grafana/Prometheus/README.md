# 📑 Panduan Instalasi & Konfigurasi **Prometheus + Grafana + Node Exporter**

Dokumentasi ini menjelaskan cara **deployment, konfigurasi, dan validasi Prometheus + Grafana menggunakan Docker Compose**, serta **Node Exporter menggunakan Docker Compose pada setiap VM target** untuk lingkungan **production / data center / enterprise**.

---

## 🧰 Prasyarat

### Server Monitoring

* OS: **Ubuntu 24.04 LTS**
* Hostname: `monitoring01`
* Akses: `root` / `sudo`
* Docker Engine sudah terinstall
* Docker Compose Plugin sudah terinstall
* CPU: minimal **4 vCPU**, direkomendasikan **8 vCPU**
* RAM: minimal **8 GB**, direkomendasikan **16 GB**
* Storage: minimal **100 GB**, direkomendasikan **300 GB SSD/NVMe**
* Dapat mengakses seluruh VM target melalui TCP `9100`

### VM Target

Setiap VM Linux yang akan dimonitor membutuhkan:

* Docker Engine sudah terinstall
* Docker Compose Plugin sudah terinstall
* Network menuju server Prometheus
* TCP `9100` dapat diakses dari server Prometheus

### Port

| Service       |       Port | Keterangan            |
| ------------- | ---------: | --------------------- |
| Grafana       | `3000/tcp` | Dashboard             |
| Prometheus    | `9090/tcp` | Prometheus Web UI/API |
| Node Exporter | `9100/tcp` | Metrics VM            |

---

# 🏗️ Arsitektur

```text
                         RSUD NETWORK
                              │
                              │
                  ┌───────────▼───────────┐
                  │     monitoring01       │
                  │     Ubuntu 24.04       │
                  │                        │
                  │     Docker Compose     │
                  │                        │
                  │  ┌──────────────────┐  │
                  │  │    Prometheus    │  │
                  │  │      :9090       │  │
                  │  └────────┬─────────┘  │
                  │           │            │
                  │  ┌────────▼─────────┐  │
                  │  │     Grafana      │  │
                  │  │      :3000       │  │
                  │  └──────────────────┘  │
                  └───────────┬────────────┘
                              │
                     Prometheus Scrape
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
   │   SIMRS VM  │     │ PostgreSQL  │     │  SNOMED VM  │
   │             │     │     VM      │     │             │
   │ Node        │     │ Node        │     │ Node        │
   │ Exporter    │     │ Exporter    │     │ Exporter    │
   │ :9100       │     │ :9100       │     │ :9100       │
   └─────────────┘     └─────────────┘     └─────────────┘
```

### Alur Data

```text
VM Target
   │
   │ Node Exporter :9100
   ▼
Prometheus :9090
   │
   │ PromQL
   ▼
Grafana :3000
   │
   ▼
Dashboard Monitoring
```

> **Catatan:** Node Exporter tidak melakukan push data ke Prometheus. Prometheus secara berkala melakukan **scrape** endpoint `/metrics` pada Node Exporter.

---

# 🖥️ BAGIAN A — SERVER MONITORING

## 🌐 STEP 1: Persiapan Directory

Buat struktur direktori:

```bash
mkdir -p /opt/monitoring/prometheus
mkdir -p /opt/monitoring/grafana/provisioning/datasources
```

Struktur:

```text
/opt/monitoring/
├── docker-compose.yml
├── prometheus/
│   └── prometheus.yml
└── grafana/
    └── provisioning/
        └── datasources/
            └── prometheus.yml
```

---

# 🐳 STEP 2: Docker Compose Prometheus + Grafana

Buat file:

```bash
nano /opt/monitoring/docker-compose.yml
```

Isi:

```yaml
services:

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped

    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=90d'
      - '--web.enable-lifecycle'

    ports:
      - "9090:9090"

    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus

    networks:
      - monitoring

    healthcheck:
      test:
        - CMD
        - wget
        - --spider
        - -q
        - http://localhost:9090/-/healthy
      interval: 30s
      timeout: 10s
      retries: 3


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
      - ./grafana/provisioning:/etc/grafana/provisioning:ro

    depends_on:
      prometheus:
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


volumes:
  prometheus_data:
  grafana_data:


networks:
  monitoring:
    driver: bridge
```

### Ganti Password Grafana

Ubah:

```yaml
GF_SECURITY_ADMIN_PASSWORD: CHANGE_THIS_STRONG_PASSWORD
```

menjadi password administrator yang kuat.

---

# 📊 STEP 3: Konfigurasi Prometheus

Buat:

```bash
nano /opt/monitoring/prometheus/prometheus.yml
```

Konfigurasi awal:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

  external_labels:
    site: rsud
    environment: production


scrape_configs:

  # Prometheus Server
  - job_name: prometheus

    static_configs:
      - targets:
          - 'prometheus:9090'

        labels:
          server: monitoring01
          role: monitoring


  # Linux Servers
  - job_name: node

    static_configs:

      - targets:
          - '10.10.1.10:9100'
        labels:
          server: simrs01
          role: simrs
          environment: production

      - targets:
          - '10.10.1.11:9100'
        labels:
          server: postgres01
          role: database
          environment: production

      - targets:
          - '10.10.1.12:9100'
        labels:
          server: snomed01
          role: snomed
          environment: production

      - targets:
          - '10.10.1.13:9100'
        labels:
          server: web01
          role: web
          environment: production
```

Sesuaikan IP dan hostname dengan environment RSUD.

---

# 📌 STEP 4: Konfigurasi Datasource Grafana

Buat:

```bash
nano /opt/monitoring/grafana/provisioning/datasources/prometheus.yml
```

Isi:

```yaml
apiVersion: 1

datasources:

  - name: Prometheus
    type: prometheus
    access: proxy

    url: http://prometheus:9090

    isDefault: true
    editable: false

    jsonData:
      httpMethod: POST
      timeInterval: 15s
```

Grafana menggunakan:

```text
http://prometheus:9090
```

karena `prometheus` adalah nama service pada Docker Compose network.

---

# 🚀 STEP 5: Deploy Prometheus + Grafana

Masuk ke directory:

```bash
cd /opt/monitoring
```

Validasi konfigurasi:

```bash
docker compose config
```

Jika tidak ada error:

```bash
docker compose up -d
```

Cek:

```bash
docker compose ps
```

Expected:

```text
NAME         STATUS
prometheus   Up
grafana      Up
```

---

# 🔍 STEP 6: Validasi Container

### Prometheus

```bash
docker logs prometheus --tail 100
```

Cek health:

```bash
docker inspect \
  --format='{{.State.Health.Status}}' \
  prometheus
```

Expected:

```text
healthy
```

### Grafana

```bash
docker logs grafana --tail 100
```

Cek:

```bash
docker inspect \
  --format='{{.State.Health.Status}}' \
  grafana
```

Expected:

```text
healthy
```

---

# 🌐 STEP 7: Akses Grafana

Buka:

```text
http://IP_MONITORING:3000
```

Contoh:

```text
http://10.10.1.50:3000
```

Login:

```text
Username : admin
Password : password yang telah dibuat
```

---

# 📈 STEP 8: Validasi Prometheus

Akses:

```text
http://IP_MONITORING:9090
```

Kemudian:

```text
Status
└── Targets
```

Untuk sementara target Prometheus akan terlihat:

```text
prometheus    UP
```

Target Node Exporter akan menjadi `UP` setelah agent selesai dipasang.

---

# 🖥️ BAGIAN B — NODE EXPORTER PADA VM TARGET

Node Exporter dipasang pada **setiap VM Linux** yang ingin dimonitor.

Contoh:

```text
SIMRS01
PostgreSQL01
SNOMED01
WEB01
API01
Docker01
Backup01
```

---

# 📁 STEP 9: Directory Node Exporter

Pada setiap VM target:

```bash
mkdir -p /opt/node-exporter
cd /opt/node-exporter
```

Buat:

```bash
nano docker-compose.yml
```

Isi:

```yaml
services:

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped

    command:
      - '--path.rootfs=/host'
      - '--collector.systemd'
      - '--collector.processes'

    pid: host

    ports:
      - "9100:9100"

    volumes:
      - '/:/host:ro,rslave'

    security_opt:
      - no-new-privileges:true
```

---

# 🚀 STEP 10: Deploy Node Exporter

Validasi:

```bash
docker compose config
```

Jalankan:

```bash
docker compose up -d
```

Cek:

```bash
docker compose ps
```

Expected:

```text
NAME            STATUS
node-exporter   Up
```

---

# 🔍 STEP 11: Test Node Exporter

Dari VM target:

```bash
curl http://127.0.0.1:9100/metrics
```

Harus muncul metric seperti:

```text
# HELP node_cpu_seconds_total Seconds the CPUs spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} ...
```

Cek port:

```bash
ss -lntp | grep 9100
```

Expected:

```text
LISTEN 0 4096 0.0.0.0:9100
```

---

# 🔐 STEP 12: Firewall Node Exporter

Misalkan IP server Prometheus:

```text
10.10.1.50
```

Pada **setiap VM target**:

```bash
ufw allow from 10.10.1.50 to any port 9100 proto tcp
```

Cek:

```bash
ufw status numbered
```

Target yang diharapkan:

```text
9100/tcp ALLOW FROM 10.10.1.50
```

> Jangan membuka port `9100` ke seluruh jaringan jika tidak diperlukan.

---

# 🧪 STEP 13: Test dari Server Prometheus

Dari `monitoring01`:

```bash
curl http://10.10.1.10:9100/metrics
```

Atau:

```bash
nc -vz 10.10.1.10 9100
```

Jika berhasil, koneksi:

```text
Prometheus
    │
    │ TCP 9100
    ▼
Node Exporter
```

sudah tersedia.

---

# 📊 STEP 14: Tambahkan VM ke Prometheus

Edit:

```bash
nano /opt/monitoring/prometheus/prometheus.yml
```

Contoh:

```yaml
- job_name: node

  static_configs:

    - targets:
        - '10.10.1.10:9100'
      labels:
        server: simrs01
        role: simrs
        environment: production

    - targets:
        - '10.10.1.11:9100'
      labels:
        server: postgres01
        role: database
        environment: production

    - targets:
        - '10.10.1.12:9100'
      labels:
        server: snomed01
        role: snomed
        environment: production
```

---

# 🔄 STEP 15: Reload Prometheus

Karena `--web.enable-lifecycle` sudah diaktifkan:

```bash
curl -X POST http://127.0.0.1:9090/-/reload
```

Atau restart:

```bash
cd /opt/monitoring
docker compose restart prometheus
```

---

# ✅ STEP 16: Validasi Target

Buka:

```text
http://10.10.1.50:9090/targets
```

Target harus menunjukkan:

```text
10.10.1.10:9100    UP
10.10.1.11:9100    UP
10.10.1.12:9100    UP
```

Jika `UP`, Node Exporter sudah berhasil dimonitor Prometheus.

---

# 📊 STEP 17: Import Dashboard Node Exporter Full

Gunakan Dashboard:

**Node Exporter Full — ID 1860**

Di Grafana:

```text
Dashboards
    ↓
New
    ↓
Import
    ↓
Import dashboard from Grafana.com
    ↓
1860
    ↓
Load
```

Pilih datasource:

```text
Prometheus
```

Kemudian:

```text
Import
```

Dashboard akan menampilkan:

```text
CPU
Memory
Disk
Filesystem
Network
Load
Uptime
I/O
Processes
```

---

# 🏷️ STEP 18: Standarisasi Label

Untuk environment dengan banyak VM, gunakan label yang konsisten.

Contoh:

```yaml
labels:
  server: simrs01
  role: simrs
  environment: production
  site: rsud
```

Contoh lengkap:

```yaml
- targets:
    - '10.10.1.10:9100'
  labels:
    server: simrs01
    role: simrs
    environment: production
    site: rsud

- targets:
    - '10.10.1.11:9100'
  labels:
    server: postgres01
    role: database
    environment: production
    site: rsud

- targets:
    - '10.10.1.12:9100'
  labels:
    server: snomed01
    role: snomed
    environment: production
    site: rsud
```

Label ini nantinya memudahkan filtering dashboard berdasarkan:

```text
Server
Role
Environment
Site
```

---

# 📁 STEP 19: Struktur Direktori

### Server Monitoring

```text
/opt/monitoring/
│
├── docker-compose.yml
│
├── prometheus/
│   └── prometheus.yml
│
└── grafana/
    └── provisioning/
        └── datasources/
            └── prometheus.yml
```

### Setiap VM Target

```text
/opt/node-exporter/
└── docker-compose.yml
```

---

# 🔄 STEP 20: Management

### Start

```bash
cd /opt/monitoring
docker compose up -d
```

### Stop

```bash
docker compose stop
```

### Restart

```bash
docker compose restart
```

### Update Image

```bash
docker compose pull
docker compose up -d
```

### Cek Container

```bash
docker compose ps
```

### Logs

```bash
docker compose logs -f
```

Prometheus:

```bash
docker compose logs -f prometheus
```

Grafana:

```bash
docker compose logs -f grafana
```

---

# 💾 STEP 21: Backup

Data utama berada di Docker volume:

```text
prometheus_data
grafana_data
```

Cek:

```bash
docker volume ls
```

Backup konfigurasi:

```bash
tar -czf /root/monitoring-config-$(date +%F).tar.gz \
    /opt/monitoring
```

Backup Grafana:

```bash
docker run --rm \
  -v monitoring_grafana_data:/data \
  -v /backup:/backup \
  alpine \
  tar czf /backup/grafana-data-$(date +%F).tar.gz -C /data .
```

Backup Prometheus:

```bash
docker run --rm \
  -v monitoring_prometheus_data:/data \
  -v /backup:/backup \
  alpine \
  tar czf /backup/prometheus-data-$(date +%F).tar.gz -C /data .
```

> Backup `/opt/monitoring` tetap wajib dilakukan karena konfigurasi Prometheus dan provisioning Grafana berada di sana.

---

# 🔐 STEP 22: Keamanan

### Prometheus

Jika Prometheus hanya diperlukan oleh administrator:

```bash
ufw allow from 10.10.1.0/24 to any port 9090 proto tcp
```

### Grafana

```bash
ufw allow from 10.10.1.0/24 to any port 3000 proto tcp
```

Sesuaikan subnet dengan network management RSUD.

### Node Exporter

Pada setiap VM:

```bash
ufw allow from 10.10.1.50 to any port 9100 proto tcp
```

Dengan:

```text
10.10.1.50 = monitoring01
```

Konsep keamanan:

```text
                   monitoring01
                  10.10.1.50
                       │
                       │ :9100
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           VM01      VM02      VM03
           :9100     :9100     :9100
```

Hanya `monitoring01` yang diperbolehkan melakukan scrape Node Exporter.

---

# 🧪 STEP 23: Troubleshooting

| Masalah                                     | Kemungkinan Penyebab      | Solusi                           |
| ------------------------------------------- | ------------------------- | -------------------------------- |
| Grafana tidak bisa dibuka                   | Container mati            | `docker compose ps`              |
| Prometheus tidak bisa dibuka                | Container mati            | `docker compose logs prometheus` |
| Target `DOWN`                               | Node Exporter tidak aktif | Cek `docker compose ps`          |
| Target `DOWN`                               | Firewall                  | Test `curl IP:9100`              |
| Target `DOWN`                               | Routing                   | Cek routing/network              |
| Metric kosong                               | Prometheus config         | Cek `prometheus.yml`             |
| Grafana tidak punya datasource              | Provisioning error        | Cek logs Grafana                 |
| Dashboard 1860 kosong                       | Datasource salah          | Pilih Prometheus                 |
| Disk Prometheus cepat penuh                 | Retention/series tinggi   | Evaluasi retention               |
| Node Exporter tidak menampilkan host metric | Mount `/host` bermasalah  | Cek volume Node Exporter         |

### Test Node Exporter

Dari `monitoring01`:

```bash
curl http://10.10.1.10:9100/metrics
```

### Cek Prometheus Target

```text
http://10.10.1.50:9090/targets
```

Semua target harus:

```text
UP
```

### Test Prometheus Query

Query:

```promql
up
```

Expected:

```text
monitoring01    1
simrs01         1
postgres01      1
snomed01        1
```

CPU:

```promql
node_cpu_seconds_total
```

Memory:

```promql
node_memory_MemAvailable_bytes
```

Filesystem:

```promql
node_filesystem_avail_bytes
```

---

# 📊 STEP 24: Pengembangan Monitoring

## Phase 1 — Infrastructure

```text
Node Exporter
│
├── CPU
├── RAM
├── Disk
├── Disk I/O
├── Network
├── Load
├── Filesystem
└── Uptime
```

## Phase 2 — PostgreSQL

```text
PostgreSQL
    │
    └── postgres_exporter
          │
          ├── Connections
          ├── Cache Hit Ratio
          ├── Transactions
          ├── Locks
          ├── Database Size
          └── Replication
```

## Phase 3 — Docker

```text
Docker
    │
    └── cAdvisor
          │
          ├── Container CPU
          ├── Container RAM
          ├── Network
          └── Container Status
```

## Phase 4 — Application

```text
Nginx
PHP-FPM
Elasticsearch
SNOMED
```

Exporter masing-masing dapat ditambahkan sesuai kebutuhan.

---

# 🏥 Arsitektur Final RSUD

```text
                                  ┌──────────────────┐
                                  │     Grafana      │
                                  │      :3000       │
                                  └────────▲─────────┘
                                           │
                                           │
                                  ┌────────┴─────────┐
                                  │    Prometheus    │
                                  │      :9090       │
                                  │                  │
                                  │ Retention: 90d   │
                                  └────────▲─────────┘
                                           │
                              scrape /metrics
                                           │
       ┌───────────────────────────────────┼─────────────────────────────┐
       │                                   │                             │
       ▼                                   ▼                             ▼
┌───────────────┐                  ┌───────────────┐             ┌───────────────┐
│   SIMRS VM    │                  │ PostgreSQL VM │             │   SNOMED VM    │
│               │                  │               │             │               │
│ Node Exporter │                  │ Node Exporter │             │ Node Exporter │
│    :9100      │                  │    :9100      │             │    :9100      │
└───────────────┘                  └───────────────┘             └───────────────┘

       ┌───────────────┐                  ┌───────────────┐
       │   WEB/API VM  │                  │   OTHER VM    │
       │               │                  │               │
       │ Node Exporter │                  │ Node Exporter │
       │    :9100      │                  │    :9100      │
       └───────────────┘                  └───────────────┘
```

Dengan arsitektur ini:

```text
                 ┌─────────────────────┐
                 │     monitoring01    │
                 │                     │
                 │ Prometheus + Grafana│
                 └──────────┬──────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
         VM1               VM2               VM3
          │                 │                 │
        Node              Node              Node
       Exporter           Exporter          Exporter
```

satu server monitoring dapat menjadi **central monitoring seluruh VM RSUD**, sementara setiap VM hanya menjalankan **Node Exporter**.

---

# 📌 Catatan Penting

> ⚠️ Perubahan konfigurasi **WAJIB diuji di staging** sebelum diterapkan ke production.

> ⚠️ Untuk production, sebaiknya image Docker tidak menggunakan tag `latest` secara permanen. Setelah deployment stabil, gunakan versi yang dipin agar proses upgrade dan rollback lebih terkontrol.

> ⚠️ Prometheus menyimpan metric secara lokal. Pastikan storage `monitoring01` memiliki kapasitas cukup dan lakukan backup konfigurasi serta data sesuai kebijakan backup RSUD.

> ⚠️ Port `9100` Node Exporter sebaiknya hanya dapat diakses oleh server Prometheus.

---

# 📎 Referensi

* [Prometheus Documentation](https://prometheus.io/docs/?utm_source=chatgpt.com)
* [Prometheus Node Exporter Guide](https://prometheus.io/docs/guides/node-exporter/?utm_source=chatgpt.com)
* [Grafana Documentation](https://grafana.com/docs/grafana/latest/?utm_source=chatgpt.com)
* [Node Exporter Full Dashboard #1860](https://grafana.com/grafana/dashboards/1860-node-exporter-full/?utm_source=chatgpt.com)
* [Docker Documentation](https://docs.docker.com/?utm_source=chatgpt.com)
