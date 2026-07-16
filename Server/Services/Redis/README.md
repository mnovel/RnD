# 📑 Panduan Instalasi & Konfigurasi **Redis Server**

Dokumentasi ini menjelaskan cara **instalasi, konfigurasi, dan validasi**
**Redis Server** pada **Ubuntu 24.04 LTS** untuk lingkungan **production / data center / enterprise**.

---

## 🧰 Prasyarat

* OS : **Ubuntu Server 24.04 LTS**
* Akses : `root` atau user dengan hak `sudo`
* Koneksi jaringan (jika Redis diakses dari server lain)
* Port service terbuka (**6379/TCP**)
* Repository Ubuntu aktif dan dapat mengakses internet

---

## 🏗️ Arsitektur (Opsional)

```text
+----------------------+
|     Redis Server     |
|    10.10.1.20:6379   |
+----------------------+
            ^
            |
---------------------------------------------
|                     |                     |
+----------------+  +----------------+  +----------------+
| Laravel Server |  | API Server     |  | Worker Server  |
| 10.10.1.30     |  | 10.10.1.31     |  | 10.10.1.32     |
+----------------+  +----------------+  +----------------+
```

---

# 🌐 STEP 1: Instalasi Redis Server

## 1.1 Update Sistem

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 1.2 Instal Redis

```bash
sudo apt install redis-server -y
```

Cek versi Redis:

```bash
redis-server --version
```

Contoh output:

```text
Redis server v=7.x.x
```

---

## 1.3 Enable & Start Service

```bash
sudo systemctl enable redis-server
sudo systemctl start redis-server
sudo systemctl status redis-server
```

Pastikan status:

```text
Active: active (running)
```

---

# 🖥️ STEP 2: Konfigurasi Server

## 2.1 File Konfigurasi Utama

Lokasi konfigurasi:

```text
/etc/redis/redis.conf
```

Edit konfigurasi:

```bash
sudo nano /etc/redis/redis.conf
```

Parameter yang umum digunakan pada server production:

### Bind Address

Agar hanya menerima koneksi dari localhost:

```conf
bind 127.0.0.1 -::1
```

Jika ingin dapat diakses server lain dalam jaringan internal:

```conf
bind 0.0.0.0
```

atau

```conf
bind 10.10.1.20
```

---

### Protected Mode

Untuk akses lokal:

```conf
protected-mode yes
```

Jika membuka akses ke jaringan internal, tetap gunakan autentikasi dan firewall.

---

### Port

```conf
port 6379
```

---

### Password (Direkomendasikan)

```conf
requirepass PasswordRedisYangKuat
```

---

### Memory Management (Opsional)

Misalnya membatasi penggunaan RAM maksimal 4 GB.

```conf
maxmemory 4gb
```

Kebijakan penghapusan data saat memori penuh:

```conf
maxmemory-policy allkeys-lru
```

---

### Persistence

Snapshot RDB:

```conf
save 900 1
save 300 10
save 60 10000
```

Append Only File (AOF):

```conf
appendonly yes
```

---

## 2.2 Restart Service

```bash
sudo systemctl restart redis-server
```

Pastikan tidak ada error:

```bash
sudo systemctl status redis-server
```

---

# 🖥️ STEP 3: Konfigurasi Client

## 3.1 Test Menggunakan redis-cli

Jika Redis menggunakan password:

```bash
redis-cli
```

Kemudian login:

```text
AUTH PasswordRedisYangKuat
```

Atau langsung:

```bash
redis-cli -a PasswordRedisYangKuat
```

Jika Redis berada di server lain:

```bash
redis-cli -h 10.10.1.20 -p 6379 -a PasswordRedisYangKuat
```

---

## 3.2 Test Operasi Dasar

Menyimpan data:

```bash
SET nama "Redis Server"
```

Mengambil data:

```bash
GET nama
```

Hasil:

```text
"Redis Server"
```

---

# 📁 STEP 4: Struktur Direktori / Data

```text
/etc/redis/
└── redis.conf

/var/lib/redis/
└── dump.rdb

/var/log/redis/
└── redis-server.log
```

---

# ✅ STEP 5: Validasi & Testing

## 5.1 Cek Status

```bash
systemctl status redis-server
```

---

## 5.2 Test Manual

Ping Redis:

```bash
redis-cli ping
```

Output:

```text
PONG
```

Jika menggunakan password:

```bash
redis-cli -a PasswordRedisYangKuat ping
```

---

## 5.3 Cek Informasi Redis

```bash
redis-cli INFO
```

Untuk melihat penggunaan memori:

```bash
redis-cli INFO memory
```

Untuk melihat statistik:

```bash
redis-cli INFO stats
```

Pastikan:

* Service **running**
* Port **6379** aktif
* Tidak ada error log
* Redis merespons perintah dengan normal

---

# 🔐 STEP 6: Keamanan

* Jangan membuka port Redis ke internet.
* Gunakan password (`requirepass`).
* Gunakan firewall untuk membatasi akses.
* Aktifkan `protected-mode yes`.
* Gunakan jaringan internal/VPN untuk komunikasi antar server.
* Jalankan Redis menggunakan user `redis` (default Ubuntu).

Contoh firewall menggunakan UFW:

Hanya mengizinkan subnet internal:

```bash
sudo ufw allow from 10.10.1.0/24 to any port 6379 proto tcp
```

Atau menolak seluruh akses eksternal:

```bash
sudo ufw deny 6379/tcp
```

---

# 📊 STEP 7: Monitoring & Logging

Monitoring dapat diintegrasikan dengan:

* Redis Exporter
* Prometheus
* Grafana
* Zabbix
* ELK / OpenSearch

Melihat penggunaan memori:

```bash
redis-cli INFO memory
```

Melihat jumlah koneksi:

```bash
redis-cli INFO clients
```

Melihat statistik server:

```bash
redis-cli INFO stats
```

Log Redis:

```text
/var/log/redis/redis-server.log
```

Pastikan log dikelola menggunakan **logrotate** agar tidak memenuhi kapasitas disk.

---

# 🧪 STEP 8: Troubleshooting

Cek log service:

```bash
sudo journalctl -u redis-server -f
```

Cek apakah Redis mendengarkan pada port 6379:

```bash
ss -tulpn | grep 6379
```

Cek koneksi:

```bash
redis-cli ping
```

Masalah umum:

| Masalah                                | Penyebab                   | Solusi                                                                                       |
| -------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------- |
| Service tidak start                    | Konfigurasi salah          | Jalankan `redis-server /etc/redis/redis.conf --test-memory 2` dan periksa syntax konfigurasi |
| Port 6379 tidak listen                 | Redis gagal start          | Cek `systemctl status redis-server` dan log service                                          |
| Tidak bisa konek dari server lain      | Firewall atau bind address | Periksa `bind`, `protected-mode`, dan aturan firewall                                        |
| Error `NOAUTH Authentication required` | Password belum diberikan   | Gunakan `AUTH <password>` atau `redis-cli -a <password>`                                     |

---

# 📌 Catatan Penting

> ⚠️ Untuk lingkungan **production**, hindari membuka port Redis ke publik. Gunakan jaringan internal, autentikasi (`requirepass` atau ACL), firewall, dan lakukan pengujian konfigurasi terlebih dahulu pada lingkungan **staging** sebelum diterapkan ke server **production**.

---

# 📎 Referensi

* [Redis Documentation](https://redis.io/docs/?utm_source=chatgpt.com)
* [Redis Security Guide](https://redis.io/docs/latest/operate/oss_and_stack/management/security/?utm_source=chatgpt.com)
* [Ubuntu Server Documentation](https://ubuntu.com/server/docs?utm_source=chatgpt.com)
