# 🐧 Panduan Instalasi & Hardening Ubuntu Server 24.04 LTS

Dokumentasi ini menjelaskan **langkah lengkap instalasi Ubuntu Server 24.04** serta **hardening dasar–menengah** untuk kebutuhan server produksi (web, aplikasi, database, logging, dsb).

Cocok untuk environment **Data Center / VM / Cloud / On‑Premise**.

---

## 🧰 Prasyarat

* ISO **Ubuntu Server 24.04 LTS**
* Akses fisik / KVM / console cloud
* Koneksi internet
* User dengan hak `sudo`

---

## 🏗️ STEP 1: Instalasi Ubuntu Server 24.04

### 1.1 Boot & Installer

1. Boot dari ISO Ubuntu Server 24.04
2. Pilih **Install Ubuntu Server**
3. Pilih:

   * Language
   * Keyboard layout
   * Timezone (Asia/Jakarta)

---

### 1.2 Network Configuration

* Gunakan **Static IP** untuk server

Contoh:

```
Address : 10.10.1.20/24
Gateway : 10.10.1.1
DNS     : 1.1.1.1, 8.8.8.8
```

---

### 1.3 Storage Configuration

Rekomendasi:

* Use **LVM** (fleksibel resize)
* Pisahkan partisi penting:

| Mount Point | Size Minimal |
| ----------- | ------------ |
| /           | 20 GB        |
| /var        | 10–20 GB     |
| /home       | Opsional     |
| swap        | 2–4 GB       |

---

### 1.4 User & SSH

* Buat user non-root (misal: `admin`)
* Aktifkan **OpenSSH Server**

---

## 🔄 STEP 2: Post-Installation Basic Setup

### 2.1 Update Sistem

```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Install Paket Dasar

```bash
sudo apt install -y \
  curl wget git vim net-tools \
  htop unzip zip ca-certificates \
  software-properties-common
```

---

## 🔐 STEP 3: Hardening Dasar (WAJIB)

### 3.1 Disable Login Root via SSH

Edit konfigurasi SSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Pastikan:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

### 3.2 Gunakan SSH Key (Client Side)

```bash
ssh-keygen -t ed25519
ssh-copy-id admin@IP_SERVER
```

---

### 3.3 Firewall (UFW)

Aktifkan firewall:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
```

Cek status:

```bash
sudo ufw status verbose
```

---

### 3.4 Fail2Ban (Brute Force Protection)

```bash
sudo apt install fail2ban -y
```

Enable:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

## 🛡️ STEP 4: Hardening Sistem Lanjutan

### 4.1 Kernel Hardening (sysctl)

Edit:

```bash
sudo nano /etc/sysctl.d/99-hardening.conf
```

Isi:

```
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
kernel.randomize_va_space = 2
fs.protected_symlinks = 1
fs.protected_hardlinks = 1
```

Apply:

```bash
sudo sysctl --system
```

---

### 4.2 Disable Unused Filesystem

Edit:

```bash
sudo nano /etc/modprobe.d/blacklist.conf
```

Tambahkan:

```
install cramfs /bin/true
install squashfs /bin/true
install udf /bin/true
```

---

### 4.3 Automatic Security Update

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

---

## 🧾 STEP 5: Logging & Audit

### 5.1 Enable Auditd

```bash
sudo apt install auditd audispd-plugins -y
sudo systemctl enable auditd
```

### 5.2 Centralized Logging (Opsional)

Gunakan:

* **RSyslog Server**
* **ELK / Graylog / SIEM**

---

## 👤 STEP 6: User & Permission Hardening

* Jangan gunakan user bersama
* Gunakan sudo terbatas:

```bash
sudo visudo
```

Contoh:

```
admin ALL=(ALL) ALL
```

---

## 🔎 STEP 7: Monitoring & Security Tools (Opsional)

Rekomendasi:

* `lynis` (security audit)
* `chkrootkit`
* `rkhunter`

```bash
sudo apt install lynis -y
sudo lynis audit system
```

---

## 🧪 STEP 8: Verifikasi Hardening

Cek:

```bash
ss -tulpn
sudo ufw status
sudo journalctl -p err
```

---

## 📌 Best Practice Tambahan

* Gunakan **2FA SSH** untuk server kritikal
* Gunakan **VPN** untuk akses admin
* Backup rutin (`rsnapshot`, `borg`)
* Monitoring resource (Prometheus, Zabbix)

---

## ✅ Checklist Produksi

* [x] SSH key only
* [x] Firewall aktif
* [x] Fail2Ban
* [x] Kernel hardening
* [x] Auto security update
* [x] Logging aktif
