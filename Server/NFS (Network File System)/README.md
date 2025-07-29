# 📁 NFS Server-Client Setup: Berbagi Folder Laravel `/var/www/be/public/storage`

Dokumentasi ini menjelaskan langkah-langkah konfigurasi NFS (Network File System) di Ubuntu untuk membagikan folder Laravel (`/var/www/be/public/storage`) dari **Server NFS** ke **Client NFS**. Folder tersebut akan dapat diakses dan ditulis oleh client.

---

## 🧹 Topologi

| Peran      | Hostname      | IP            |
| ---------- | ------------- | ------------- |
| NFS Server | `backend-a`   | `172.16.1.96` |
| NFS Client | `webserver-a` | `172.16.1.95` |

* **Folder dibagikan**: `/var/www/be/public/storage`
* **Lokasi mount di client**: `/var/www/be/public/storage`

---

## 🔧 1. Konfigurasi di Server (172.16.1.96 - NFS Server)

### ✅ Install NFS Server

```bash
sudo apt update
sudo apt install nfs-kernel-server
```

---

### ✅ Persiapan Folder Laravel

```bash
php artisan storage:link
sudo chown -R www-data:www-data /var/www/be/public/storage
sudo chmod -R 755 /var/www/be/public/storage
```

---

### ✅ Konfigurasi Ekspor NFS

Edit file konfigurasi:

```bash
sudo nano /etc/exports
```

Tambahkan baris berikut:

```
/var/www/be/public/storage 172.16.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```

> Catatan: `no_root_squash` mengizinkan client untuk menulis sebagai root. Gunakan dengan hati-hati.

---

### ✅ Terapkan Konfigurasi Ekspor

```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
showmount -e
```

Output yang diharapkan:

```
Export list for 172.16.1.96:
/var/www/be/public/storage 172.16.1.0/24
```

---

## 💻 2. Konfigurasi di Client (172.16.1.95 - NFS Client)

### ✅ Install NFS Client

```bash
sudo apt update
sudo apt install nfs-common
```

---

### ✅ Buat Folder Mount Point

> Kita akan mount ke folder yang sama seperti di server agar Laravel dapat langsung menggunakan hasil share.

```bash
sudo mkdir -p /var/www/be/public/storage
```

---

### ✅ Mount Folder dari Server

```bash
sudo mount -t nfs 172.16.1.96:/var/www/be/public/storage /var/www/be/public/storage
```

---

### ✅ Verifikasi Akses

```bash
ls -la /var/www/be/public/storage
sudo touch /var/www/be/public/storage/test.txt
```

Jika berhasil membuat file, berarti mount sukses dan dapat ditulis.

---

### 🔁 (Opsional) Mount Otomatis Saat Booting

Edit file `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Tambahkan baris berikut di akhir file:

```
172.16.1.96:/var/www/be/public/storage /var/www/be/public/storage nfs defaults 0 0
```

Simpan dan uji:

```bash
sudo mount -a
```

---

## ✅ Tips Tambahan

* Pastikan **permission** user Laravel (`www-data`) sama di kedua server (UID dan GID).
* Jika menggunakan firewall, pastikan port **2049 (NFS)** terbuka.
* Gunakan `no_root_squash` hanya untuk pengujian atau jika benar-benar diperlukan.
