# HARDENING.md — Windows 10 (Production)

## 1. Tujuan

Dokumen ini berisi langkah **hardening keamanan Windows 10** untuk meminimalkan risiko:

- Akses tidak sah
- Malware / ransomware
- Penyalahgunaan RDP
- Kebocoran data internal

Digunakan setelah **INSTALL.md** selesai.

---

## 2. User & Account Hardening

### 2.1 Local Account

- Gunakan **Local Account**, bukan Microsoft Account
- Nonaktifkan akun yang tidak digunakan
- Ganti nama default Administrator (opsional)

Cek user:

```powershell
net user
```

---

### 2.2 Password Policy

Rekomendasi:

- Minimum 12 karakter
- Kombinasi huruf besar, kecil, angka, simbol
- Password expiration aktif (opsional)

Set via:

```
secpol.msc > Account Policies > Password Policy
```

---

## 3. Windows Update & Patch

- Aktifkan **Automatic Updates**
- Jangan menunda update security

Cek status:

```powershell
Get-WindowsUpdateLog
```

---

## 4. Windows Defender Hardening

### 4.1 Pastikan Aktif

Aktifkan:

- Real-time protection
- Cloud-delivered protection
- Automatic sample submission
- Tamper Protection

Path:

```
Windows Security > Virus & threat protection
```

---

### 4.2 Defender Exclusions (Jika Aplikasi Internal)

Tambahkan **hanya jika diperlukan**, misalnya:

- Folder aplikasi internal
- Folder data besar

⚠️ Jangan mengecualikan seluruh drive.

---

## 5. Firewall Hardening

### 5.1 Status Firewall

Pastikan **ON** untuk semua profile:

- Domain
- Private
- Public

Cek:

```powershell
Get-NetFirewallProfile
```

---

### 5.2 Inbound Rules

Best practice:

- Block semua inbound default
- Allow hanya:

  - RDP (jika digunakan)
  - Aplikasi internal
  - Monitoring agent

---

## 6. Remote Desktop (RDP) Hardening

### 6.1 Enable RDP (Jika Diperlukan)

```
Settings > System > Remote Desktop
```

---

### 6.2 Batasi User RDP

Hanya user tertentu:

```
System Properties > Remote > Select Users
```

---

### 6.3 RDP Security

Rekomendasi:

- Ubah port RDP (opsional)
- Aktifkan Network Level Authentication (NLA)
- Batasi IP via firewall

---

## 7. Service Hardening

### 7.1 Disable Service Tidak Diperlukan

Contoh service yang **umumnya bisa dinonaktifkan** (sesuaikan kebutuhan):

- Xbox Services
- Bluetooth (jika tidak digunakan)
- Print Spooler (jika tidak ada printer)
- Remote Registry

Cek:

```powershell
services.msc
```

---

## 8. Startup & Scheduled Task

### 8.1 Startup Apps

- Disable aplikasi startup tidak perlu

Path:

```
Task Manager > Startup
```

---

### 8.2 Scheduled Tasks

- Review scheduled task mencurigakan
- Disable task yang tidak dikenal

---

## 9. File System & Permission

### 9.1 Folder Aplikasi

- Batasi permission folder aplikasi
- Hindari **Everyone : Full Control**

---

### 9.2 Shared Folder (Jika Ada)

- Gunakan user khusus
- Disable anonymous access

---

## 10. Network Hardening

### 10.1 SMB Hardening

- Disable SMBv1

Cek:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

Disable:

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

---

### 10.2 Network Discovery

- Disable jika tidak diperlukan

```
Control Panel > Network and Sharing Center
```

---

## 11. Browser Hardening

Rekomendasi:

- Gunakan Chrome / Firefox terbaru
- Disable extension tidak perlu
- Aktifkan auto update browser

---

## 12. Logging & Audit

### 12.1 Event Viewer

Aktifkan logging untuk:

- Login success / failure
- System error
- Application crash

Path:

```
Event Viewer > Windows Logs
```

---

### 12.2 Time Sync

Pastikan waktu sinkron:

```powershell
w32tm /query /status
```

---

## 13. Backup & Recovery

Rekomendasi:

- Backup minimal:

  - Data penting
  - Konfigurasi aplikasi

- Simpan backup di media terpisah / network

---

## 14. Physical Security

- Aktifkan auto-lock screen
- Disable boot dari USB (jika memungkinkan)
- Password BIOS / UEFI (opsional)

---

## 15. Checklist Hardening

- [ ] Local user digunakan
- [ ] Password policy diterapkan
- [ ] Windows Update aktif
- [ ] Defender aktif & updated
- [ ] Firewall aktif
- [ ] RDP dibatasi
- [ ] Service tidak perlu dinonaktifkan
- [ ] SMBv1 disabled
- [ ] Backup tersedia
