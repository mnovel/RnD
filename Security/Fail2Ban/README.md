# 📑 Panduan Instalasi & Konfigurasi **Fail2Ban**

Dokumentasi ini menjelaskan cara **instalasi, konfigurasi, dan hardening dasar**
**Fail2Ban** pada **Ubuntu Server** untuk melindungi server dari **brute-force attack**
(SSH, web, service lain) di lingkungan **production / data center / enterprise**.

---

## 🧰 Prasyarat

- OS : **Ubuntu Server 22.04 / 24.04 LTS**
- Akses : `root` / `sudo`
- OpenSSH Server aktif
- Firewall aktif (UFW / iptables)
- Log service tersedia (`/var/log/auth.log`, dll)

---

## 🏗️ Arsitektur

```
+-----------------------+
|   Server Ubuntu       |
|   SSH / Web Service   |
|   Fail2Ban            |
+-----------------------+
            |
   Monitoring log
            |
+-----------------------+
| /var/log/auth.log     |
| /var/log/nginx/*.log  |
+-----------------------+
```

Fail2Ban bekerja **lokal di setiap server** (agent-based),
tidak membutuhkan server terpusat.

---

## 🌐 STEP 1: Instalasi Fail2Ban

### 1.1 Update Sistem

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2 Install Fail2Ban

```bash
sudo apt install fail2ban -y
```

### 1.3 Enable & Start Service

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```

---

## 🖥️ STEP 2: Konfigurasi Dasar Fail2Ban

⚠️ **JANGAN mengedit `jail.conf` langsung**

### 2.1 Buat file konfigurasi lokal

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Edit:

```bash
sudo nano /etc/fail2ban/jail.local
```

---

### 2.2 Konfigurasi Global (Rekomendasi)

```
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5

backend  = systemd
ignoreip = 127.0.0.1/8 ::1
```

Penjelasan singkat:

- **bantime** : lama IP diblokir
- **findtime** : window waktu deteksi
- **maxretry** : jumlah gagal sebelum ban
- **backend** : gunakan systemd (lebih stabil)

---

## 🔐 STEP 3: Proteksi SSH (WAJIB)

Aktifkan jail SSH:

```
[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
backend = systemd
```

📌 Jika SSH pakai port custom:

```
port = 2222
```

---

## 🌐 STEP 4: Proteksi Service Tambahan (Opsional)

### 4.1 Nginx (HTTP Auth / Basic / Login)

```
[nginx-http-auth]
enabled = true
```

### 4.2 Nginx Bot / Bad Request

```
[nginx-botsearch]
enabled = true
```

### 4.3 Custom Service

Buat filter di:

```
/etc/fail2ban/filter.d/<service>.conf
```

---

## 🔄 STEP 5: Restart & Validasi

### 5.1 Restart Fail2Ban

```bash
sudo systemctl restart fail2ban
```

### 5.2 Cek Status Jail

```bash
sudo fail2ban-client status
```

Contoh output:

```
Status
|- Number of jail: 1
`- Jail list: sshd
```

Detail jail:

```bash
sudo fail2ban-client status sshd
```

---

## 📁 STEP 6: Struktur Direktori Fail2Ban

```
/etc/fail2ban/
├── jail.conf        (default - jangan edit)
├── jail.local       (utama)
├── jail.d/
│   └── custom.local
├── filter.d/
│   ├── sshd.conf
│   └── nginx-*.conf
```

---

## 🔥 STEP 7: Integrasi Firewall

Fail2Ban otomatis terintegrasi dengan:

- **UFW**
- **iptables / nftables**

Cek action:

```
banaction = ufw
```

Atau default:

```
banaction = iptables-multiport
```

---

## 🧪 STEP 8: Testing Manual

Simulasi gagal login SSH dari IP lain:

```bash
ssh wronguser@server-ip
```

Cek IP terblokir:

```bash
sudo iptables -L -n
```

Atau:

```bash
sudo fail2ban-client get sshd banned
```

---

## 🧪 STEP 9: Troubleshooting

### Cek log Fail2Ban

```bash
sudo journalctl -u fail2ban -f
```

Atau:

```bash
sudo tail -f /var/log/fail2ban.log
```

Masalah umum:

| Masalah              | Penyebab         | Solusi             |
| -------------------- | ---------------- | ------------------ |
| Jail tidak aktif     | Salah config     | Cek syntax         |
| IP tidak ter-ban     | Log path salah   | Sesuaikan log      |
| Service restart loop | jail.conf diedit | Gunakan jail.local |

---

## 📌 Catatan Keamanan Penting

> ⚠️ Fail2Ban **BUKAN pengganti firewall**
>
> Gunakan bersama:
>
> - SSH key-only authentication
> - Disable root login
> - Firewall policy ketat
