# 📑 Panduan Instalasi & Konfigurasi **WAHA (WhatsApp HTTP API - Core)**

Dokumentasi ini menjelaskan cara **deployment, konfigurasi, dan validasi**
**WAHA (WhatsApp HTTP API - Core)** menggunakan **Docker Compose**
pada **Ubuntu 22.04 / 24.04 LTS** untuk lingkungan **production / data center / enterprise**.

---

## 🧰 Prasyarat

* OS : **Ubuntu 22.04 / 24.04 LTS**
* Akses : `root` / `sudo`
* Docker & Docker Compose sudah tersedia
* Minimal resource:

  * CPU: **2 Core**
  * RAM: **2 GB (disarankan 4 GB)**
* Port: **3000/tcp**
* Public IP / akses internal network
* Storage untuk session WA

---

## 🏗️ Arsitektur

```
+------------------------+
|     WAHA SERVER        |
|   127.0.0.1:3000       |
+------------------------+
           ^
           |
-------------------------------
|            |               |
+---------+  +---------+   +---------+
| Backend |  | Web App|   | Service |
| API     |  | Admin  |   | Worker  |
+---------+  +---------+   +---------+
```

---

## 🌐 STEP 1: Deployment WAHA

### 1.1 Buat Direktori Project

```bash
mkdir -p /opt/waha
cd /opt/waha
```

---

### 1.2 Download Template Docker Compose

```bash
wget -O docker-compose.yaml https://raw.githubusercontent.com/devlikeapro/waha/refs/heads/core/docker-compose.yaml
touch .env
```

---

### 1.3 Pastikan Image WAHA Core

Edit file:

```bash
nano docker-compose.yaml
```

Pastikan menggunakan **WAHA Core (bukan plus)**:

```yaml
services:
  waha:
    restart: always
    image: devlikeapro/waha
```

---

### 1.4 Generate File `.env`

```bash
docker compose run --no-deps -v "$(pwd)":/app/env waha init-waha /app/env
```

Akan menghasilkan kredensial:

* `WAHA_DASHBOARD_USERNAME`
* `WAHA_DASHBOARD_PASSWORD`
* `WHATSAPP_SWAGGER_USERNAME`
* `WHATSAPP_SWAGGER_PASSWORD`
* `WAHA_API_KEY`

👉 Simpan dengan aman

---

### 1.5 Hardening Credential (WAJIB)

Generate key:

```bash
uuidgen | tr -d '-'
```

Edit `.env`:

```bash
nano .env
```

Ganti:

```
WAHA_API_KEY=
WAHA_DASHBOARD_PASSWORD=
WHATSAPP_SWAGGER_PASSWORD=
```

Dengan string random kuat

---

### 1.6 Jalankan Service

```bash
docker compose up -d
```

---

## 🖥️ STEP 2: Konfigurasi Server

### 2.1 Binding Port (Default Aman)

Default:

```
127.0.0.1:3000
```

👉 Artinya **tidak expose ke publik**

---

### 2.2 (Opsional) Expose ke Network

Edit:

```bash
nano docker-compose.yaml
```

Ubah:

```yaml
ports:
  - "3000:3000"
```

Restart:

```bash
docker compose up -d
```

---

### 2.3 Environment Penting

```env
WAHA_API_KEY=
WAHA_BASE_URL=
WAHA_LOG_LEVEL=info
```

---

## 🖥️ STEP 3: Akses & Client

### 3.1 Dashboard

```
http://localhost:3000/dashboard
```

---

### 3.2 Swagger API

```
http://localhost:3000/swagger
```

---

### 3.3 Endpoint API

```
http://<IP_SERVER>:3000
```

Header:

```
X-Api-Key: <WAHA_API_KEY>
```

---

## 📁 STEP 4: Struktur Direktori

```
/opt/waha/
├── docker-compose.yaml
├── .env
├── sessions/
└── logs/
```

---

## ✅ STEP 5: Validasi & Testing

### 5.1 Cek Container

```bash
docker ps
```

---

### 5.2 Cek Log

```bash
docker compose logs -f
```

---

### 5.3 Test API

```bash
curl http://localhost:3000/api/sessions
```

---

### 5.4 Login WhatsApp

1. Akses dashboard
2. Scan QR
3. Status → **connected**

---

## 🔐 STEP 6: Keamanan

### Best Practice:

* Jangan expose langsung ke internet
* Gunakan reverse proxy
* Gunakan HTTPS
* Gunakan API key kuat

---

### Firewall

```bash
ufw allow 3000/tcp
```

---

## 🌐 STEP 7: Reverse Proxy (Nginx)

### 7.1 Install & Config

```bash
apt install nginx
cd /etc/nginx/sites-enabled
nano waha.conf
```

---

### 7.2 Config

```nginx
server {
  server_name <DOMAIN_OR_IP>;

  set $upstream 127.0.0.1:3000;

  location / {
    proxy_pass http://$upstream;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $host;

    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_http_version 1.1;
    proxy_read_timeout 36000s;
  }

  listen 80;
}
```

---

### 7.3 Apply Config

```bash
nginx -t
systemctl reload nginx
```

---

## 🔒 STEP 8: HTTPS (Let's Encrypt)

```bash
apt install certbot python3-certbot-nginx
certbot --nginx -d <DOMAIN>
```

---

### Update `.env`

```
WAHA_BASE_URL=https://<DOMAIN>
```

Restart:

```bash
docker compose restart
```

---

## 🔄 STEP 9: Operasional

### Update WAHA

```bash
docker compose pull
docker compose up -d
```

---

### Restart

```bash
docker compose restart
```

---

### Stop

```bash
docker compose down
```

---

### Logs

```bash
docker compose logs -f
```

---

## 🧪 STEP 10: Troubleshooting

```bash
docker compose logs -f
```

---

### Masalah Umum

| Masalah           | Penyebab      | Solusi                |
| ----------------- | ------------- | --------------------- |
| Tidak bisa akses  | Port binding  | Cek docker-compose    |
| QR tidak muncul   | Session error | Hapus folder sessions |
| API unauthorized  | API key salah | Cek `.env`            |
| Container restart | Config error  | Validasi YAML         |

---

## 📌 Catatan Penting

> ⚠️ Default WAHA hanya bind ke localhost (lebih aman)

> ⚠️ Jangan gunakan credential lemah

> ⚠️ Logout WhatsApp di HP akan memutus session

---

## 📎 Referensi

* WAHA GitHub (Core Branch)
* Docker Documentation
* Best Practice Reverse Proxy & API Security

