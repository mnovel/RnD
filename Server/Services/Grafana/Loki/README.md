# 📑 Panduan Instalasi & Konfigurasi Grafana + Loki + Promtail (Docker Compose)

Dokumentasi ini menjelaskan proses instalasi, konfigurasi, dan validasi **Grafana, Loki, dan Promtail** menggunakan **Docker Compose** pada **Ubuntu Server 24.04 LTS** untuk lingkungan **production**.

---

# 🖥️ BAGIAN 1 — HOST (Grafana + Loki)

Host berfungsi sebagai **server monitoring** sekaligus **log aggregation server**.

---

## 🧰 Prasyarat

* Ubuntu Server 24.04 LTS
* Docker Engine
* Docker Compose Plugin
* Hak akses `sudo`
* Port yang digunakan:

  * **3000/TCP** (Grafana)
  * **3100/TCP** (Loki)

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
          ------------------------------------------
          |                  |                     |
+----------------+  +----------------+  +----------------+
| Promtail Node1 |  | Promtail Node2 |  | Promtail NodeN |
+----------------+  +----------------+  +----------------+
```

---

# 🌐 STEP 1 : Install Docker

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2 -y
```

Enable Docker

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verifikasi

```bash
docker --version
docker compose version
```

---

# 🌐 STEP 2 : Membuat Direktori

```bash
mkdir -p ~/grafana-loki/loki
mkdir -p ~/grafana-loki/grafana
cd ~/grafana-loki
```

---

# 🌐 STEP 3 : Konfigurasi Loki

Buat file

```bash
nano loki/config.yaml
```

Isi konfigurasi dasar

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /loki

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
  filesystem:
    directory: /loki/chunks

limits_config:
  allow_structured_metadata: false

ruler:
  alertmanager_url: http://localhost:9093
```

---

# 🌐 STEP 4 : Docker Compose

Buat file

```bash
nano docker-compose.yml
```

```yaml
version: "3.9"

services:

  loki:
    image: grafana/loki:latest
    container_name: loki
    restart: unless-stopped

    ports:
      - "3100:3100"

    volumes:
      - ./loki/config.yaml:/etc/loki/config.yaml
      - loki-data:/loki

    command:
      - -config.file=/etc/loki/config.yaml

  grafana:
    image: grafana/grafana:latest
    container_name: grafana

    restart: unless-stopped

    ports:
      - "3000:3000"

    volumes:
      - grafana-data:/var/lib/grafana

    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  loki-data:
  grafana-data:
```

---

# 🌐 STEP 5 : Menjalankan Container

```bash
docker compose up -d
```

Verifikasi

```bash
docker ps
```

---

# 🌐 STEP 6 : Konfigurasi Grafana

Buka

```
http://IP_SERVER:3000
```

Login

```
admin
admin
```

Tambah Data Source

```
Connections
→ Add Data Source
→ Loki
```

URL

```
http://loki:3100
```

Klik

```
Save & Test
```

---

# 📁 STEP 7 : Struktur Direktori

```text
grafana-loki/

├── docker-compose.yml

├── grafana/

├── loki/
│   └── config.yaml
```

---

# ✅ STEP 8 : Validasi

```bash
docker ps
```

Cek Loki

```bash
curl http://localhost:3100/ready
```

Output

```
ready
```

---

# 💻 BAGIAN 2 — AGENT (Promtail)

Promtail membaca log lokal kemudian mengirimkannya ke Loki.

---

## 🧰 Prasyarat

* Docker Engine
* Docker Compose
* Akses ke Host Loki
* Port 3100 dapat diakses

---

# 🌐 STEP 1 : Membuat Direktori

```bash
mkdir -p ~/promtail
cd ~/promtail
```

---

# 🌐 STEP 2 : Konfigurasi Promtail

```bash
nano config.yaml
```

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

---

# 🌐 STEP 3 : Docker Compose

```bash
nano docker-compose.yml
```

```yaml
version: "3.9"

services:

  promtail:

    image: grafana/promtail:latest

    container_name: promtail

    restart: unless-stopped

    volumes:

      - ./config.yaml:/etc/promtail/config.yaml

      - /var/log:/var/log:ro

      - /etc/machine-id:/etc/machine-id:ro

    command:

      - -config.file=/etc/promtail/config.yaml
```

---

# 🌐 STEP 4 : Menjalankan Promtail

```bash
docker compose up -d
```

---

# 🌐 STEP 5 : Validasi

Container

```bash
docker ps
```

Log

```bash
docker logs -f promtail
```

Tes koneksi

```bash
curl http://IP_SERVER:3100/ready
```

---

# 🌐 STEP 6 : Verifikasi di Grafana

Masuk ke menu:

```
Explore
```

Pilih **Loki** kemudian jalankan query:

```logql
{job="syslog"}
```

Jika log muncul, maka Promtail berhasil mengirimkan log ke Loki.

---

# 🔐 STEP 7 : Keamanan

* Batasi akses port **3000** dan **3100** menggunakan firewall.
* Gunakan *reverse proxy* (Nginx atau Traefik) dengan HTTPS untuk akses Grafana pada lingkungan produksi.
* Ganti kredensial bawaan Grafana (`admin/admin`) setelah instalasi.
* Gunakan *Docker volumes* untuk menyimpan data Grafana dan Loki secara persisten.
* Lakukan pencadangan (*backup*) terhadap volume dan berkas konfigurasi secara berkala.

---

# 📊 STEP 8 : Monitoring & Logging

Melihat status container:

```bash
docker ps
```

Melihat log Grafana:

```bash
docker logs -f grafana
```

Melihat log Loki:

```bash
docker logs -f loki
```

Melihat log Promtail:

```bash
docker logs -f promtail
```
