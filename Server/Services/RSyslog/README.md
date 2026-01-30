# 📑 Panduan Instalasi & Konfigurasi RSyslog

## Centralized Logging (Server & Client) + NGINX (Tag-Based via imfile)

Dokumentasi ini menjelaskan setup **centralized logging** menggunakan **RSyslog** pada **Ubuntu 24.04**, dengan pemisahan **NGINX access & error log** berbasis **TAG** menggunakan **imfile** di sisi client (tanpa mengubah `nginx.conf`).

---

## 🧰 Prasyarat

* Ubuntu 24.04 LTS (Server & Client)
* Akses `sudo`
* Koneksi jaringan antar node
* Firewall terbuka pada **TCP 514**
* Paket `rsyslog-modules` terpasang di **client** (untuk `imfile`)

---

## 🏗️ Arsitektur

```
          +-------------------------+
          |   RSyslog Server        |
          |   10.10.1.15            |
          +-------------------------+
                     ^
                     |  TCP 514
    -------------------------------------------
    |                     |                   |
+-----------+       +-----------+       +-----------+
| Client A  |       | Client B  |  ...  | Client N  |
|           |       |           |       |           |
+-----------+       +-----------+       +-----------+
```

---

## 🌐 STEP 1: Instalasi RSyslog (Server & Client)

### 1.1 Update sistem

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2 Instal RSyslog

```bash
sudo apt install rsyslog -y
```

### 1.3 Aktifkan service

```bash
sudo systemctl enable rsyslog
sudo systemctl restart rsyslog
```

---

## 🖥️ STEP 2: Konfigurasi RSyslog **Server** (10.10.1.15)

### 2.1 Aktifkan penerimaan TCP

Edit `/etc/rsyslog.conf`:

```conf
module(load="imtcp")
input(type="imtcp" port="514")
```

Restart:

```bash
sudo systemctl restart rsyslog
```

---

### 2.2 Konfigurasi Template & Rules

Buat file `/etc/rsyslog.d/01-custom.conf`:

```conf
############################
# TEMPLATE
############################

template(name="AuthLogs" type="string"
 string="/var/log/rsyslog/%HOSTNAME%/auth/%PROGRAMNAME%.log")

template(name="NginxAccessLogs" type="string"
 string="/var/log/rsyslog/%HOSTNAME%/nginx/access.log")

template(name="NginxErrorLogs" type="string"
 string="/var/log/rsyslog/%HOSTNAME%/nginx/error.log")

template(name="SystemLogs" type="string"
 string="/var/log/rsyslog/%HOSTNAME%/system/%PROGRAMNAME%.log")

template(name="OtherLogs" type="string"
 string="/var/log/rsyslog/%HOSTNAME%/other.log")

############################
# RULES
############################

# SSH & sudo
if ($programname == 'sshd' or $programname == 'sudo') then {
  action(type="omfile" dynaFile="AuthLogs" createDirs="on")
  stop
}

# NGINX ACCESS (TAG-BASED)
if ($programname == 'nginx_access') then {
  action(type="omfile" dynaFile="NginxAccessLogs" createDirs="on")
  stop
}

# NGINX ERROR (TAG-BASED)
if ($programname == 'nginx_error') then {
  action(type="omfile" dynaFile="NginxErrorLogs" createDirs="on")
  stop
}

# Kernel & systemd
if ($syslogfacility-text == 'kern' or $programname == 'systemd') then {
  action(type="omfile" dynaFile="SystemLogs" createDirs="on")
  stop
}

# Sisanya
*.* action(type="omfile" dynaFile="OtherLogs" createDirs="on")
```

Restart:

```bash
sudo systemctl restart rsyslog
```

---

## 🖥️ STEP 3: Konfigurasi RSyslog **Client** (web-simrs-1)

### 3.1 Pastikan NGINX menulis log ke file (default)

```nginx
access_log /var/log/nginx/access.log;
error_log  /var/log/nginx/error.log;
```

> **Tidak perlu mengubah `nginx.conf` untuk syslog.**

---

### 3.2 Konfigurasi imfile untuk NGINX (TAG-BASED)

Buat `/etc/rsyslog.d/30-nginx.conf`:

```conf
module(load="imfile")

# NGINX ACCESS
input(type="imfile"
      File="/var/log/nginx/access.log"
      Tag="nginx_access"
      Severity="info"
      Facility="local6"
      PersistStateInterval="1")

# NGINX ERROR
input(type="imfile"
      File="/var/log/nginx/error.log"
      Tag="nginx_error"
      Severity="error"
      Facility="local6"
      PersistStateInterval="1")
```

**Catatan:** `Tag` akan menjadi `$programname` di server.

---

### 3.3 Forward log ke server

Pastikan `/etc/rsyslog.d/90-forward.conf`:

```conf
*.* @@10.10.1.15:514
```

Restart:

```bash
sudo systemctl restart rsyslog
```

---

### 3.4 Permission file log (WAJIB)

```bash
sudo chown root:adm /var/log/nginx/*.log
sudo chmod 640 /var/log/nginx/*.log
```

---

## 📁 STEP 4: Verifikasi

### 4.1 Generate traffic

```bash
curl http://localhost
```

### 4.2 Cek di server

```bash
ls -R /var/log/rsyslog/web-simrs-1/nginx
```

Hasil yang diharapkan:

```
access.log
error.log
```

Tail:

```bash
tail -f /var/log/rsyslog/web-simrs-1/nginx/access.log
```

---

## 🔐 STEP 5: Firewall

**Server**:

```bash
sudo ufw allow 514/tcp
```

---

## 🧪 STEP 6: Troubleshooting

* Cek error rsyslog client:

```bash
journalctl -u rsyslog -n 50 --no-pager
```

* Test TAG manual:

```bash
logger -t nginx_access "TEST ACCESS"
logger -t nginx_error "TEST ERROR"
```

* Catatan penting:

  * `imfile` hanya membaca **log baru** (setelah restart)
  * Hindari `copytruncate` pada logrotate (gunakan `create`)
