# Windows Documentation Index

Dokumentasi ini berisi panduan **instalasi, hardening, dan troubleshooting** untuk berbagai versi **Microsoft Windows** yang digunakan pada lingkungan **production, enterprise, dan operasional internal**.

Struktur ini dirancang agar:

- Mudah diaudit
- Konsisten antar versi OS
- Siap digunakan oleh tim **DevOps, SysAdmin, dan Cyber Security**

---

## 📁 Struktur Direktori

```text
Windows/
├── 10/
│   ├── INSTALL.md
│   ├── HARDENING.md
│   └── TROUBLESHOOTING.md
│
├── 11/
│   ├── INSTALL.md
│   ├── HARDENING.md
│   └── TROUBLESHOOTING.md
│
└── README.md
```

---

## 📌 Versi Windows yang Didukung

### 🔹 Windows 10 Pro

- Status: **Stable / Production Ready**
- Support hingga: **Oktober 2025**
- Digunakan untuk:

  - Client Production
  - Sistem Monitoring
  - Aplikasi internal
  - Kiosk / Display System
  - VM operasional

📂 Lokasi:

```text
Windows/10/
```

---

### 🔹 Windows 11 Pro

- Status: **Current & Future Standard**
- Support aktif: **Yes**
- Digunakan untuk:

  - Deployment baru
  - Endpoint modern
  - High Security Environment
  - Sistem dengan TPM & Secure Boot
  - Client enterprise

📂 Lokasi:

```text
Windows/11/
```

---

## 📄 Standar Dokumen

Setiap versi Windows **WAJIB** memiliki file berikut:

| File               | Deskripsi                                |
| ------------------ | ---------------------------------------- |
| INSTALL.md         | Panduan instalasi OS & konfigurasi dasar |
| HARDENING.md       | Security hardening sesuai best practice  |
| TROUBLESHOOTING.md | Masalah umum Windows & solusi teknis     |

---

## 🔐 Security & Compliance

Dokumentasi hardening mengacu pada:

- CIS Benchmark Windows 10 & Windows 11
- Microsoft Security Baseline
- Principle of Least Privilege
- Windows Defender & Firewall Best Practice
- Audit & Event Logging (Event Viewer)

> ⚠️ **Catatan:**
> Beberapa konfigurasi hardening dapat memengaruhi:
>
> - RDP
> - Aplikasi legacy
> - Software internal
>
> **Wajib diuji di staging / test environment sebelum production.**

---

## 🛠️ Target Pengguna

Dokumentasi ini ditujukan untuk:

- DevOps Engineer
- System Administrator
- Network Engineer
- Cyber Security Engineer
- IT Infrastructure Team
- Helpdesk Level 2–3

---

## 🚀 Rencana Pengembangan

- [ ] Template Windows Kiosk / Display
- [ ] CIS Benchmark Mapping Detail
- [ ] PowerShell Auto Hardening Script
- [ ] Group Policy (GPO) Reference
- [ ] Diagram Arsitektur Endpoint
- [ ] Integrasi ke SIEM / Logging Server
