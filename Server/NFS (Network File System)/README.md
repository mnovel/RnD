# 📁 NFS Server-Client Setup: Berbagi Folder Melalui Network File System

Dokumentasi ini menjelaskan langkah-langkah konfigurasi NFS (Network File System) di Ubuntu untuk membagikan folder dari **Server NFS** ke **Client NFS**.

---

## 🧹 Topologi Umum

| Peran      | Contoh Hostname | Contoh IP      |
| ---------- | --------------- | -------------- |
| NFS Server | `server-nfs`    | `192.168.1.10` |
| NFS Client | `client-nfs`    | `192.168.1.20` |

* **Folder yang akan dibagikan (server)**: `/data/shared`
* **Folder mount di client**: `/mnt/shared`

---

## 🔧 STEP 1: Konfigurasi di NFS Server

### ✅ 1.1 Install NFS Server

```bash
sudo apt update
sudo apt install nfs-kernel-server
```

### ✅ 1.2 Siapkan Folder yang Akan Dibagikan

```bash
sudo mkdir -p /data/shared
sudo chown -R nobody:nogroup /data/shared
sudo chmod 755 /data/shared
```

> Sesuaikan permission sesuai kebutuhan.

### ✅ 1.3 Konfigurasi File Ekspor

```bash
sudo nano /etc/exports
```

Tambahkan:

```
/data/shared 192.168.1.0/24(rw,sync,no_subtree_check)
```

### ✅ 1.4 Terapkan Konfigurasi

```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
showmount -e
```

---

## 💻 STEP 2: Konfigurasi di NFS Client

### ✅ 2.1 Install NFS Client

```bash
sudo apt update
sudo apt install nfs-common
```

### ✅ 2.2 Buat Folder untuk Mount Point

```bash
sudo mkdir -p /mnt/shared
```

### ✅ 2.3 Mount Folder dari Server

```bash
sudo mount -t nfs 192.168.1.10:/data/shared /mnt/shared
```

### ✅ 2.4 Verifikasi Akses

```bash
ls -la /mnt/shared
sudo touch /mnt/shared/test.txt
```

---

## 🔁 STEP 3 (Opsional): Mount Otomatis Saat Booting

### ✅ 3.1 Edit File `/etc/fstab`

```bash
sudo nano /etc/fstab
```

Tambahkan:

```
192.168.1.10:/data/shared /mnt/shared nfs defaults 0 0
```

### ✅ 3.2 Jalankan

```bash
sudo mount -a
```

---

## ✅ Tips dan Best Practice

- Gunakan subnet yang tepat di file `/etc/exports` untuk membatasi akses client.
- Gunakan `rw,sync,no_subtree_check` sebagai opsi umum.
- Hindari `no_root_squash` kecuali benar-benar dibutuhkan.
- Pastikan UID dan GID user yang mengakses sama di kedua sisi (contoh: `www-data` = UID 33).
- Pastikan port **2049** terbuka jika menggunakan firewall.
- Untuk keamanan lebih lanjut, gunakan firewall, VPN, atau IP whitelisting.

---

## 📚 Referensi Tambahan

- [https://wiki.archlinux.org/title/NFS](https://wiki.archlinux.org/title/NFS)
- [https://help.ubuntu.com/community/NFS](https://help.ubuntu.com/community/NFS)
