# 🪟 Panduan Instalasi & Hardening Windows 10 / Windows 11 (Non-Server)

Dokumentasi ini menjelaskan **instalasi, konfigurasi dasar, dan hardening keamanan Windows 10 / Windows 11** yang digunakan sebagai **endpoint / workstation produksi** (kantor, admin IT, NOC, SOC, operator SIMRS, dsb).

⚠️ **Bukan Windows Server** dan **tidak menggunakan Active Directory secara mandatory**.

---

## 🧰 Prasyarat

* ISO **Windows 10 / Windows 11 (64-bit)** resmi
* Lisensi Windows valid
* Akses Administrator lokal
* Koneksi internet

---

## 🏗️ STEP 1: Instalasi Windows

### 1.1 Instalasi Dasar

1. Boot dari installer Windows
2. Pilih:

   * Language & region: **Indonesia**
   * Keyboard: **US / Indonesia**
3. Pilih **Custom Install**
4. Hapus partisi lama (jika clean install)
5. Install ke disk utama

---

### 1.2 Akun Awal

Rekomendasi:

* **Gunakan Local Account** (bukan Microsoft Account) untuk workstation kritikal
* Username contoh: `admin.local`

---

## 🔄 STEP 2: Post-Installation Basic Setup

### 2.1 Windows Update

* Jalankan **Windows Update sampai clean**
* Aktifkan auto-update security

```text
Settings → Windows Update → Advanced Options
```

---

### 2.2 Install Software Wajib

Rekomendasi minimal:

* Google Chrome / Firefox
* 7-Zip
* Notepad++
* Sysinternals Suite
* Antivirus (Defender default sudah cukup)

---

## 🔐 STEP 3: Hardening Dasar (WAJIB)

### 3.1 User Account Control (UAC)

Aktifkan UAC maksimal:

```text
Control Panel → User Accounts → Change UAC settings
```

Set ke:

```
Always notify
```

---

### 3.2 Pisahkan Akun Admin & User

Best practice:

* 1 akun **Admin lokal** (tidak dipakai harian)
* 1 akun **User standar** untuk operasional

```powershell
net user admin-secure /add
net localgroup administrators admin-secure /add
```

---

### 3.3 Password Policy

Buka:

```text
secpol.msc
```

Set:

* Minimum password length: **12**
* Password complexity: **Enabled**
* Maximum password age: **90 days**

---

## 🛡️ STEP 4: Windows Defender Hardening

### 4.1 Pastikan Defender Aktif

```text
Windows Security → Virus & Threat Protection
```

Aktifkan:

* Real-time protection
* Cloud-delivered protection
* Automatic sample submission

---

### 4.2 Enable Tamper Protection

```text
Windows Security → Virus & Threat Protection → Manage Settings
```

Set:

```
Tamper Protection: ON
```

---

### 4.3 Attack Surface Reduction (ASR)

Buka:

```text
Windows Security → App & Browser Control → Exploit Protection
```

Rekomendasi:

* Enable **DEP, ASLR**
* Enable **Controlled Folder Access**

---

## 🔥 STEP 5: Firewall Hardening

### 5.1 Windows Defender Firewall

Pastikan aktif untuk:

* Domain
* Private
* Public

```text
Windows Security → Firewall & Network Protection
```

---

### 5.2 Outbound Control (Opsional – Advanced)

* Default: Allow
* Block aplikasi tidak dikenal

```text
wf.msc
```

---

## 🧱 STEP 6: System Hardening

### 6.1 Disable SMBv1

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

---

### 6.2 Disable Unused Services

Contoh:

* Remote Registry
* Fax
* Xbox Services (jika workstation kantor)

```text
services.msc
```

---

### 6.3 BitLocker (WAJIB untuk Laptop)

```text
Settings → Privacy & Security → Device Encryption
```

Pastikan:

* BitLocker ON
* Recovery key dibackup aman

---

## 🧾 STEP 7: Logging & Audit

### 7.1 Enable Advanced Audit Policy

```text
secpol.msc → Advanced Audit Policy
```

Enable:

* Logon / Logoff
* Account Lockout
* Process Creation

---

### 7.2 PowerShell Logging

```powershell
Set-ItemProperty HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging -Name EnableScriptBlockLogging -Value 1
```

---

## 🔍 STEP 8: Application & Browser Hardening

### 8.1 Browser Security

* Disable password saving
* Enable HTTPS only
* Install uBlock Origin

---

### 8.2 Macro Protection (Office)

```text
Office → Trust Center → Macro Settings
```

Set:

```
Disable all macros with notification
```

---

## 🧪 STEP 9: Verification & Monitoring

Cek:

* Windows Security status
* Event Viewer → Security
* Firewall rules

Tool audit:

* Microsoft Defender Offline Scan
* Sysmon (advanced)

---

## 📌 Best Practice Tambahan

* Jangan login admin untuk kerja harian
* Lock screen otomatis (≤ 5 menit)
* Backup rutin (OneDrive / NAS / external)
* VPN untuk akses sistem internal

---

## ✅ Checklist Endpoint Produksi

* [x] Local account
* [x] UAC max
* [x] Defender aktif
* [x] Firewall aktif
* [x] SMBv1 off
* [x] BitLocker (laptop)
* [x] Audit log aktif
