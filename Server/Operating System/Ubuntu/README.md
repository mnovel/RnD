# Ubuntu Server Documentation Index

Dokumentasi ini berisi panduan **instalasi, hardening, dan troubleshooting** untuk berbagai versi **Ubuntu Server** yang digunakan pada lingkungan **production, data center, dan enterprise**.

Struktur ini dirancang agar:

- Mudah diaudit
- Konsisten antar versi OS
- Siap digunakan oleh tim **DevOps, SysAdmin, dan Cyber Security**

---

## 📁 Struktur Direktori

```
Ubuntu/
├── 22.04/
│   ├── INSTALL.md
│   ├── HARDENING.md
│   └── TROUBLESHOOTING.md
│
├── 24.04/
│   ├── INSTALL.md
│   ├── HARDENING.md
│   └── TROUBLESHOOTING.md
│
└── README.md
```

---

## 📌 Versi Ubuntu yang Didukung

### 🔹 Ubuntu Server 22.04 LTS (Jammy Jellyfish)

- Status: **Stable / Production Ready**
- Support hingga: **2032 (ESM)**
- Digunakan untuk:

  - VM Production
  - Server legacy
  - Aplikasi enterprise stabil

📂 Lokasi:

```
Ubuntu/22.04/
```

---

### 🔹 Ubuntu Server 24.04 LTS (Noble Numbat)

- Status: **Latest LTS**
- Support hingga: **2034 (ESM)**
- Digunakan untuk:

  - Deployment baru
  - Container Host
  - Kubernetes Node
  - High Security Environment

📂 Lokasi:

```
Ubuntu/24.04/
```

---

## 📄 Standar Dokumen

Setiap versi Ubuntu **WAJIB** memiliki file berikut:

| File               | Deskripsi                                |
| ------------------ | ---------------------------------------- |
| INSTALL.md         | Panduan instalasi OS & konfigurasi dasar |
| HARDENING.md       | Security hardening sesuai best practice  |
| TROUBLESHOOTING.md | Masalah umum & solusi                    |

---

## 🔐 Security & Compliance

Dokumentasi hardening mengacu pada:

- CIS Benchmark Ubuntu Server
- Best Practice SSH & Firewall
- Principle of Least Privilege
- Audit & Logging (systemd / journald)

> ⚠️ **Catatan:**
> Beberapa konfigurasi hardening dapat memengaruhi kompatibilitas aplikasi.
> Selalu uji di **staging environment** sebelum production.

---

## 🛠️ Target Pengguna

Dokumentasi ini ditujukan untuk:

- DevOps Engineer
- System Administrator
- Network Engineer
- Cyber Security Engineer
- IT Infrastructure Team

---

## 🚀 Rencana Pengembangan

- [ ] Tambah Ubuntu 20.04 (Legacy)
- [ ] CIS Benchmark Mapping Detail
- [ ] Bash Auto Hardening Script
- [ ] Ansible Role Ubuntu Hardening
- [ ] Diagram Arsitektur

---
