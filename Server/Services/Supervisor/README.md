# 📑 Panduan Instalasi & Konfigurasi **Supervisor**

Dokumentasi ini menjelaskan cara **instalasi, konfigurasi, dan validasi**
**Supervisor** pada **Ubuntu 24.04 LTS** untuk lingkungan **production / data center / enterprise**.

---

## 🧰 Prasyarat

* OS : **Ubuntu 24.04 LTS**
* Akses : `root` / `sudo`
* Python3 (sudah tersedia pada Ubuntu)
* Koneksi internet untuk instalasi package
* Port tambahan **tidak diperlukan** (Supervisor berjalan secara lokal)

---

## 🏗️ Arsitektur (Opsional)

```
+--------------------------------+
|        Ubuntu Server           |
|                                |
|        Supervisor              |
|              |                 |
|   -------------------------    |
|   |          |            |    |
| Worker 1  Worker 2   Worker N  |
| (Service) (Service) (Service)  |
+--------------------------------+
```

Supervisor bertugas mengelola proses aplikasi agar selalu berjalan (process manager), seperti:

* Laravel Queue Worker
* Python Script
* NodeJS Service
* Custom Daemon

---

# 🌐 STEP 1: Instalasi Supervisor

## 1.1 Update Sistem

```bash
sudo apt update
sudo apt upgrade -y
```

## 1.2 Instal Supervisor

```bash
sudo apt install supervisor -y
```

## 1.3 Enable & Start Service

```bash
sudo systemctl enable supervisor
sudo systemctl start supervisor
sudo systemctl status supervisor
```

Output yang diharapkan:

```
Active: active (running)
```

---

# 🖥️ STEP 2: Konfigurasi Server

## 2.1 File Konfigurasi Utama

Lokasi konfigurasi:

```
/etc/supervisor/
```

File utama:

```
/etc/supervisor/supervisord.conf
```

Direktori konfigurasi program:

```
/etc/supervisor/conf.d/
```

---

### Contoh Konfigurasi Laravel Queue

Buat file:

```bash
sudo nano /etc/supervisor/conf.d/laravel-worker.conf
```

Isi konfigurasi:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d

command=/usr/bin/php /var/www/project/artisan queue:work --sleep=3 --tries=2 --timeout=300

directory=/var/www/project

autostart=true
autorestart=true
stopasgroup=true
killasgroup=true

user=www-data

numprocs=4

redirect_stderr=true

stdout_logfile=/var/log/supervisor/laravel-worker.log
stopwaitsecs=3600
```

---

## 2.2 Reload Konfigurasi

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

Restart worker:

```bash
sudo supervisorctl restart laravel-worker:*
```

---

# 🖥️ STEP 3: Manajemen Service

Melihat seluruh service:

```bash
sudo supervisorctl status
```

Menjalankan service:

```bash
sudo supervisorctl start laravel-worker:*
```

Menghentikan service:

```bash
sudo supervisorctl stop laravel-worker:*
```

Restart service:

```bash
sudo supervisorctl restart laravel-worker:*
```

Masuk ke console Supervisor:

```bash
sudo supervisorctl
```

---

# 📁 STEP 4: Struktur Direktori

```
/etc/supervisor/
├── supervisord.conf
└── conf.d/
    ├── laravel-worker.conf
    ├── python-app.conf
    └── node-app.conf

/var/log/supervisor/
├── supervisord.log
├── laravel-worker.log
└── supervisor.log
```

---

# ✅ STEP 5: Validasi & Testing

## 5.1 Cek Status Supervisor

```bash
sudo systemctl status supervisor
```

---

## 5.2 Cek Worker

```bash
sudo supervisorctl status
```

Contoh output:

```
laravel-worker:laravel-worker_00   RUNNING
laravel-worker:laravel-worker_01   RUNNING
laravel-worker:laravel-worker_02   RUNNING
laravel-worker:laravel-worker_03   RUNNING
```

---

## 5.3 Reload Konfigurasi

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

---

## 5.4 Melihat Log

```bash
tail -f /var/log/supervisor/laravel-worker.log
```

Pastikan:

* Supervisor **running**
* Semua worker **RUNNING**
* Tidak ada konfigurasi yang gagal dimuat
* Tidak terdapat error pada log

---

# 🔐 STEP 6: Keamanan

* Jalankan proses menggunakan user non-root (`www-data` atau user aplikasi).
* Batasi hak akses file konfigurasi.

```bash
chmod 640 /etc/supervisor/conf.d/*.conf
```

* Pastikan hanya administrator yang dapat mengubah konfigurasi.
* Hindari menjalankan service sebagai `root` jika tidak diperlukan.
* Lakukan backup konfigurasi sebelum perubahan.

---

# 📊 STEP 7: Monitoring & Logging

Supervisor menyediakan log untuk setiap proses yang dikelola.

Lokasi log:

```
/var/log/supervisor/
```

Monitoring dapat diintegrasikan dengan:

* RSyslog
* ELK / OpenSearch
* Prometheus Node Exporter
* Grafana
* Zabbix

Pastikan log menggunakan mekanisme rotasi agar tidak memenuhi kapasitas penyimpanan.

---

# 🧪 STEP 8: Troubleshooting

Cek status Supervisor:

```bash
sudo systemctl status supervisor
```

Reload konfigurasi:

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

Restart seluruh proses:

```bash
sudo supervisorctl restart all
```

Melihat log Supervisor:

```bash
journalctl -u supervisor -f
```

Melihat log aplikasi:

```bash
tail -f /var/log/supervisor/laravel-worker.log
```

Masalah umum:

| Masalah                           | Penyebab                    | Solusi                                                         |
| --------------------------------- | --------------------------- | -------------------------------------------------------------- |
| Service Supervisor tidak berjalan | Konfigurasi rusak           | Periksa `supervisord.conf` dan jalankan `supervisorctl reread` |
| Program berstatus `FATAL`         | Command salah               | Pastikan path executable dan parameter benar                   |
| Program sering restart            | Aplikasi crash              | Periksa log aplikasi untuk mengetahui penyebab                 |
| Worker tidak muncul               | File `.conf` belum dimuat   | Jalankan `supervisorctl reread` dan `update`                   |
| Permission denied                 | Hak akses user tidak sesuai | Periksa nilai `user=` dan permission direktori aplikasi        |

---

# 📌 Catatan Penting

> ⚠️ Setiap perubahan konfigurasi Supervisor **WAJIB diuji pada lingkungan staging** sebelum diterapkan ke **production**. Setelah menambahkan atau mengubah file konfigurasi, selalu jalankan `supervisorctl reread` dan `supervisorctl update` agar perubahan diterapkan.

---

# 📎 Referensi

* Dokumentasi resmi Supervisor
* Ubuntu Server Guide
* Best Practice Process Management Linux
* Dokumentasi Laravel Queue (jika digunakan untuk worker Laravel)
