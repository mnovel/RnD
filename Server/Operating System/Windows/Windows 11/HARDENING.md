# HARDENING.md — Windows 11 (Production)

## 1. Tujuan

Dokumen ini menjelaskan **langkah hardening keamanan Windows 11** untuk sistem produksi guna:

- Mengurangi attack surface
- Mencegah akses tidak sah
- Menjaga stabilitas & integritas sistem

Dilakukan setelah **INSTALL.md**.

---

## 2. Account & Authentication Hardening

### 2.1 User Account

- Gunakan **Local Account**
- Nonaktifkan user tidak digunakan
- Ganti nama default Administrator (opsional)

Cek user:

```powershell
net user
```

---

### 2.2 Password Policy

Rekomendasi:

- Minimum 12 karakter
- Complexity enabled
- Lockout policy aktif

Path:

```
secpol.msc > Account Policies
```

---

## 3. Windows Update & Patch Management

- Automatic Update **ON**
- Jangan menunda security update

Cek status:

```powershell
Get-HotFix
```

---

## 4. Windows Security / Defender Hardening

### 4.1 Microsoft Defender

Pastikan aktif:

- Real-time protection
- Cloud-delivered protection
- Tamper Protection
- Controlled Folder Access (opsional)

Path:

```
Windows Security > Virus & threat protection
```

---

### 4.2 Defender Exclusion

Tambahkan **hanya jika diperlukan** untuk aplikasi internal.
⚠️ Jangan exclude seluruh drive.

---

## 5. Firewall Hardening

### 5.1 Firewall Status

Pastikan semua profile **ON**:

```powershell
Get-NetFirewallProfile
```

---

### 5.2 Firewall Rules

Best practice:

- Default: Block inbound
- Allow hanya:

  - RDP (jika digunakan)
  - Aplikasi internal
  - Monitoring agent

---

## 6. Remote Desktop (RDP) Hardening

### 6.1 RDP Configuration

- Aktifkan **Network Level Authentication (NLA)**
- Batasi user RDP

Path:

```
Settings > System > Remote Desktop
```

---

### 6.2 RDP Protection

Rekomendasi:

- Batasi IP via firewall
- Ubah port RDP (opsional)
- Disable RDP jika tidak digunakan

---

## 7. Service & Feature Hardening

### 7.1 Disable Service Tidak Digunakan

Contoh umum:

- Xbox Services
- Bluetooth (jika tidak digunakan)
- Print Spooler (jika tidak ada printer)
- Remote Registry

Cek:

```powershell
services.msc
```

---

### 7.2 Windows Features

- Disable SMBv1

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

---

## 8. Network Hardening

- Disable Network Discovery (jika tidak diperlukan)
- Nonaktifkan file sharing publik
- Pastikan DNS trusted

---

## 9. Application & Startup Hardening

### 9.1 Startup Apps

- Disable startup tidak perlu

```
Task Manager > Startup
```

---

### 9.2 Scheduled Tasks

- Review scheduled task mencurigakan
- Disable task yang tidak dikenal

---

## 10. Logging & Audit

- Aktifkan logging login success & failure
- Review Event Viewer berkala

```
Event Viewer > Windows Logs
```

---

## 11. Backup & Recovery

Rekomendasi:

- Backup data penting
- Backup konfigurasi aplikasi
- Simpan di media terpisah / network

---

## 12. Physical & Boot Security

- Auto-lock screen aktif
- Disable boot USB (jika memungkinkan)
- Password BIOS / UEFI
- Secure Boot aktif (default Windows 11)

---

## 13. Checklist Hardening

- [ ] Local account digunakan
- [ ] Password policy aktif
- [ ] Windows Update aktif
- [ ] Defender aktif & updated
- [ ] Firewall aktif
- [ ] RDP dibatasi / disabled
- [ ] SMBv1 disabled
- [ ] Backup tersedia
