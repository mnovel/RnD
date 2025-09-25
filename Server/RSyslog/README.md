# 📑 Panduan Instalasi & Konfigurasi RSyslog (Server & Client)

Dokumentasi ini menjelaskan cara setup **centralized logging** menggunakan **RSyslog** pada Ubuntu 24.04.
Arsitektur ini memungkinkan semua server (client) mengirim log ke **server pusat** (10.10.1.15).

---

## 🧰 Prasyarat

* Ubuntu 24.04 LTS (Server & Client)
* Akses `sudo`
* Koneksi jaringan antar node
* Firewall terbuka pada port **UDP/TCP 514**

---

## 🏗️ Arsitektur

```
          +--------------------+
          |   RSyslog Server   |
          |   10.10.1.15       |
          +--------------------+
                   ^
                   |
    ---------------------------------
    |               |               |
+-----------+  +-----------+  +-----------+
| Client 1  |  | Client 2  |  | Client N  |
|10.10.1.20 |  |10.10.1.21 |  |10.10.1.x  |
+-----------+  +-----------+  +-----------+
```

---

## 🌐 STEP 1: Instalasi RSyslog

### 1.1 Update sistem

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2 Instal RSyslog

```bash
sudo apt install rsyslog -y
```

### 1.3 Pastikan service berjalan

```bash
sudo systemctl enable rsyslog
sudo systemctl start rsyslog
sudo systemctl status rsyslog
```

---

## 🖥️ STEP 2: Konfigurasi RSyslog Server (10.10.1.15)

### 2.1 Edit konfigurasi rsyslog

```bash
sudo nano /etc/rsyslog.conf
```

Aktifkan modul UDP & TCP (uncomment):

```
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")
```

### 2.2 Buat folder log untuk tiap client

```bash
sudo mkdir -p /var/log/rsyslog/web-simrs-1/nginx
sudo mkdir -p /var/log/rsyslog/web-simrs-2/nginx
```

### 2.3 Buat konfigurasi custom

```bash
sudo nano /etc/rsyslog.d/01-custom.conf
```

Isi:

```
# Template simpan log per host & app
template(name="PerHostLogs" type="string" string="/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log")

# Simpan semua log client
*.* ?PerHostLogs
```

### 2.4 Restart service

```bash
sudo systemctl restart rsyslog
```

---

## 🖥️ STEP 3: Konfigurasi RSyslog Client (10.10.1.20)

### 3.1 Edit konfigurasi default

```bash
sudo nano /etc/rsyslog.d/50-default.conf
```

Tambahkan di akhir file:

```
# Kirim semua log ke server (UDP & TCP)
*.* @10.10.1.15:514
*.* @@10.10.1.15:514
```

### 3.2 Tambah custom Nginx log

```bash
sudo nano /etc/rsyslog.d/30-nginx.conf
```

Isi:

```
# Kirim log Nginx access & error
if ($programname == 'nginx') then {
    action(type="omfwd" target="10.10.1.15" port="514" protocol="tcp")
}
```

### 3.3 Restart service

```bash
sudo systemctl restart rsyslog
```

---

## 📁 STEP 4: Verifikasi Log

### 4.1 Dari sisi server

```bash
ls -R /var/log/rsyslog/
```

Contoh hasil:

```
/var/log/rsyslog/web-simrs-1:
syslog.log
nginx.log
auth.log
```

### 4.2 Cek isi log

```bash
tail -f /var/log/rsyslog/web-simrs-1/nginx.log
```

---

## 🔐 STEP 5: Keamanan

1. **Firewall (server)**
   Buka port 514:

   ```bash
   sudo ufw allow 514/tcp
   sudo ufw allow 514/udp
   ```

2. **TLS/SSL (opsional)**
   Bisa menggunakan `gtls` module pada rsyslog untuk enkripsi.

---

## 📊 STEP 6: Integrasi SIEM / ELK

* Install **Filebeat** untuk forward log ke Elasticsearch
* Atau integrasi langsung dengan **Graylog / Splunk**

---

## 🧪 STEP 7: Troubleshooting

* Cek error rsyslog:

  ```bash
  journalctl -u rsyslog -f
  ```
* Test manual:

  ```bash
  logger "Test log from client"
  ```

---

## 📌 Catatan Tambahan

* Gunakan **@@ (TCP)** untuk log penting → lebih andal.
* Gunakan **@ (UDP)** untuk log ringan → lebih cepat, tapi tidak terjamin.
* Struktur direktori bisa disesuaikan per aplikasi / per host.

