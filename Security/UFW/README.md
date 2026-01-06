# 📑 Panduan Instalasi & Konfigurasi **UFW (Uncomplicated Firewall)**

Dokumentasi ini menjelaskan cara **instalasi, konfigurasi, dan validasi**
**UFW (Uncomplicated Firewall)** pada **Ubuntu Server 22.04 / 24.04 LTS**
untuk lingkungan **production / data center / enterprise**.

---

## 🧰 Prasyarat

- OS : **Ubuntu Server 22.04 / 24.04 LTS**
- Akses : `root` / `sudo`
- Koneksi jaringan aktif
- Firewall port policy telah ditentukan (SSH, HTTP, HTTPS, dsb)
- Service jaringan sudah teridentifikasi

---

## 🏗️ Arsitektur (Opsional)

```
+-----------------------+
|   Ubuntu Server       |
|   UFW Firewall        |
|   10.10.1.10          |
+-----------------------+
          |
   Filtering Traffic
          |
+-----------------------+
|   Network / Internet  |
+-----------------------+
```

UFW bekerja **langsung di host**, memfilter traffic **incoming & outgoing**
menggunakan backend **iptables / nftables**.

---

## 🌐 STEP 1: Instalasi **UFW**

### 1.1 Update Sistem

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2 Instal Service

```bash
sudo apt install ufw -y
```

### 1.3 Enable & Start Service

```bash
sudo systemctl enable ufw
sudo ufw enable
sudo ufw status verbose
```

⚠️ **Pastikan rule SSH sudah ditambahkan sebelum enable UFW**

---

## 🖥️ STEP 2: Konfigurasi Server

### 2.1 Konfigurasi Dasar Firewall

Set default policy:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Izinkan SSH:

```bash
sudo ufw allow ssh
```

Atau port custom:

```bash
sudo ufw allow 2222/tcp
```

---

### 2.2 Buka Port Service Umum

HTTP / HTTPS:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Custom service:

```bash
sudo ufw allow 514/tcp
sudo ufw allow 514/udp
```

---

### 2.3 Batasi Akses Berdasarkan IP (Rekomendasi)

```bash
sudo ufw allow from 10.10.1.0/24 to any port 22 proto tcp
```

Blokir IP:

```bash
sudo ufw deny from 203.0.113.10
```

---

### 2.4 Reload Firewall

```bash
sudo ufw reload
```

---

## 📁 STEP 4: Struktur Direktori / Data

```
/etc/ufw/
├── ufw.conf
├── before.rules
├── after.rules
└── applications.d/

/var/log/
└── ufw.log
```

---

## ✅ STEP 5: Validasi & Testing

### 5.1 Cek Status

```bash
sudo ufw status numbered
```

### 5.2 Test Manual

Dari client lain:

```bash
ssh user@server-ip
nc -zv server-ip 80
```

Pastikan:

- Rule aktif dan berurutan dengan benar
- Port yang tidak diizinkan **tertutup**
- SSH tetap dapat diakses

---

## 🔐 STEP 6: Keamanan

- Terapkan **default deny**
- Batasi akses berdasarkan IP
- Gunakan Fail2Ban untuk proteksi brute-force
- Aktifkan logging firewall

Aktifkan logging:

```bash
sudo ufw logging on
```

---

## 📊 STEP 7: Monitoring & Logging

- Log UFW tersedia di:

  ```
  /var/log/ufw.log
  ```

- Integrasi dengan:

  - RSyslog (centralized logging)
  - SIEM / ELK Stack
  - Fail2Ban (ban otomatis)

---

## 🧪 STEP 8: Troubleshooting

Cek log firewall:

```bash
sudo tail -f /var/log/ufw.log
```

Masalah umum:

| Masalah                 | Penyebab            | Solusi                    |
| ----------------------- | ------------------- | ------------------------- |
| SSH terblokir           | Rule belum dibuat   | Allow SSH sebelum enable  |
| Port tidak bisa diakses | Rule salah / urutan | Cek `ufw status numbered` |
| Rule tidak aktif        | Belum reload        | Jalankan `ufw reload`     |

---

## 📌 Catatan Penting

> ⚠️ Kesalahan konfigurasi firewall dapat menyebabkan **server tidak dapat diakses**
> Selalu:
>
> - Tambahkan rule SSH terlebih dahulu
> - Gunakan console / out-of-band access saat enable pertama kali

---

## 📎 Referensi

- Official Documentation UFW
- Ubuntu Security Guide
- CIS Benchmark Linux Firewall
