# INSTALL.md — Windows 10 (Production)

## 1. Informasi Umum

- OS : Windows 10 Pro (64-bit)
- Arsitektur : amd64 (x64)
- Tipe : VM / Bare Metal
- Role : Production System / Client / Monitoring

---

## 2. Instalasi OS

### 2.1 Konfigurasi Awal

- Language : English (United States) / Indonesia
- Region : Indonesia
- Timezone : (UTC+07:00) Bangkok, Hanoi, Jakarta
- Keyboard : US

---

### 2.2 Network Configuration (Static IP – Rekomendasi)

Gunakan IP statis untuk sistem produksi.

Contoh:

```
IP Address : 10.10.1.40
Subnet     : 255.255.255.0
Gateway    : 10.10.1.1
DNS        : 1.1.1.1, 8.8.8.8
```

📌 Lokasi konfigurasi:

```
Control Panel > Network and Sharing Center > Change adapter settings
```

---

### 2.3 Storage Configuration

Rekomendasi partisi:

| Drive | Fungsi           | Size    |
| ----- | ---------------- | ------- |
| C:\   | OS & Aplikasi    | ≥ 80 GB |
| D:\   | Data / Log / App | ≥ 50 GB |

Catatan:

- File system: **NTFS**
- Pisahkan OS dan data bila memungkinkan
- BitLocker **tidak direkomendasikan** untuk sistem produksi non-enterprise

---

### 2.4 User & Remote Access

- Buat **Local User** (non-Microsoft Account)
- User memiliki hak **Administrator**
- Contoh user: `admin`

Remote Access:

- RDP **opsional**
- Batasi user RDP
- Batasi IP melalui Windows Firewall jika diaktifkan

---

## 3. Post Installation

### 3.1 Windows Update

Pastikan sistem ter-update penuh:

```
Settings > Update & Security > Windows Update
Status : You're up to date
```

---

### 3.2 Install Paket / Software Dasar

Rekomendasi software dasar:

- Google Chrome / Mozilla Firefox
- 7-Zip
- Notepad++
- Git
- PowerShell 7
- OpenSSH Client

---

### 3.3 Enable OpenSSH Client (Opsional)

Aktifkan melalui:

```
Settings > Apps > Optional Features > OpenSSH Client
```

Verifikasi:

```powershell
ssh -V
```

---

## 4. Validasi Awal

Jalankan melalui **PowerShell**:

```powershell
systeminfo
ipconfig
Get-NetIPAddress
```

Pastikan:

- IP sesuai konfigurasi
- Network aktif
- Sistem berjalan normal
