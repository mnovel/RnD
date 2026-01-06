# INSTALL.md — Ubuntu Server 24.04 LTS

## 1. Informasi Umum

- OS : Ubuntu Server 24.04 LTS (Noble Numbat)
- Arsitektur : amd64
- Tipe : VM / Bare Metal
- Role : Production Server

---

## 2. Instalasi OS

### 2.1 Konfigurasi Awal

- Language : English / Indonesia
- Timezone : Asia/Jakarta
- Keyboard : US

---

### 2.2 Network Configuration (Static IP – Rekomendasi)

Gunakan IP statis untuk server produksi.

Contoh:

```
IP Address : 10.10.1.30/24
Gateway    : 10.10.1.1
DNS        : 1.1.1.1, 8.8.8.8
```

---

### 2.3 Storage Configuration

Rekomendasi partisi:

| Mount Point | Size    |
| ----------- | ------- |
| /           | ≥ 25 GB |
| /var        | ≥ 15 GB |
| swap        | 2–4 GB  |

Gunakan **LVM** untuk fleksibilitas resize.

---

### 2.4 User & SSH

- Buat user non-root (contoh: `admin`)
- Aktifkan **OpenSSH Server** saat instalasi

---

## 3. Post Installation

Update sistem:

```bash
sudo apt update && sudo apt upgrade -y
```

Install paket dasar:

```bash
sudo apt install -y \
  curl wget vim git htop \
  net-tools unzip zip ca-certificates \
  software-properties-common
```

---

## 4. Validasi Awal

```bash
lsb_release -a
ip a
systemctl status ssh
```
