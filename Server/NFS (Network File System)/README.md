# 📁 NFS Server-Client Setup: Berbagi Folder Melalui Network File System

Dokumentasi ini menjelaskan langkah-langkah konfigurasi NFS (Network File System) di Ubuntu untuk membagikan folder dari **Server NFS** ke **Client NFS**. Folder yang dibagikan akan dapat diakses dan ditulis oleh client, yang berguna untuk berbagai kebutuhan seperti berbagi file antar server, menyatukan storage, dan kebutuhan sinkronisasi data.

---

## 🧹 Topologi Umum

| Peran      | Contoh Hostname | Contoh IP      |
| ---------- | --------------- | -------------- |
| NFS Server | `server-nfs`    | `192.168.1.10` |
| NFS Client | `client-nfs`    | `192.168.1.20` |

* **Folder yang akan dibagikan (server)**: `/data/shared`
* **Folder mount di client**: `/mnt/shared`

---

## 🔧 1. Konfigurasi di NFS Server

### ✅ Install NFS Server

```bash
sudo apt update
sudo apt install nfs-kernel-server
```

---

### ✅ Siapkan Folder yang Akan Dibagikan

```bash
sudo mkdir -p /data/shared
sudo chown -R nobody:nogroup /data/shared
sudo chmod 755 /data/shared
```

> Sesuaikan permission sesuai kebutuhan. Bisa diganti dengan user spesifik seperti `www-data` jika digunakan untuk web.

---

### ✅ Konfigurasi File Ekspor

Edit file berikut:

```bash
sudo nano /etc/exports
```

Tambahkan:

```
/data/shared 192.168.1.0/24(rw,sync,no_subtree_check)
```

> Ganti `192.168.1.0/24` dengan subnet IP client Anda.

---

### ✅ Terapkan Konfigurasi

```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
showmount -e
```

Output contoh:

```
Export list for server-nfs:
/data/shared 192.168.1.0/24
```

---

## 💻 2. Konfigurasi di NFS Client

### ✅ Install NFS Client

```bash
sudo apt update
sudo apt install nfs-common
```

---

### ✅ Buat Folder untuk Mount Point

```bash
sudo mkdir -p /mnt/shared
```

---

### ✅ Mount Folder dari Server

```bash
sudo mount -t nfs 192.168.1.10:/data/shared /mnt/shared
```

---

### ✅ Verifikasi Akses

```bash
ls -la /mnt/shared
sudo touch /mnt/shared/test.txt
```

Jika berhasil membuat file, berarti folder berhasil di-mount dan dapat ditulis.

---

### 🔁 (Opsional) Mount Otomatis Saat Booting

Edit file `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Tambahkan baris berikut:

```
192.168.1.10:/data/shared /mnt/shared nfs defaults 0 0
```

Kemudian jalankan:

```bash
sudo mount -a
```

---

## ✅ Tips dan Best Practice

* Gunakan subnet yang tepat di file `/etc/exports` untuk membatasi akses client.
* Gunakan `rw,sync,no_subtree_check` sebagai opsi umum. Hindari `no_root_squash` kecuali benar-benar dibutuhkan.
* Pastikan UID dan GID user yang mengakses sama di kedua server (misal `www-data` = UID 33).
* Pastikan port **2049** terbuka jika menggunakan firewall.
* Untuk keamanan lebih lanjut, pertimbangkan penggunaan firewall, VPN, atau IP whitelisting.

---

## 📚 Referensi Tambahan

* [https://wiki.archlinux.org/title/NFS](https://wiki.archlinux.org/title/NFS)
* [https://help.ubuntu.com/community/NFS](https://help.ubuntu.com/community/NFS)

---