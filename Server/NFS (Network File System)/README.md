# 📁 Panduan Konfigurasi NFS di Ubuntu 24.04

Dokumentasi lengkap untuk membagikan folder antara server dan client menggunakan Network File System (NFS).

---

## 🧰 Prasyarat
- Ubuntu 24.04 LTS di server dan client
- Akses user dengan hak `sudo` di kedua sisi
- Koneksi jaringan yang stabil
- Firewall mengizinkan traffic NFS (port 2049)

---

## 🌐 Topologi Jaringan
| Peran       | Contoh Hostname | Contoh IP       |
|-------------|-----------------|-----------------|
| **NFS Server** | `server-nfs`    | `192.168.1.10`  |
| **NFS Client** | `client-nfs`    | `192.168.1.20`  |

*Folder yang dibagikan*: `/data/shared`  
*Mount point client*: `/mnt/shared`

---

## 🖥️ STEP 1: Konfigurasi NFS Server

### 1.1 Install Paket NFS Server
```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

### 1.2 Siapkan Direktori Shared
```bash
sudo mkdir -p /data/shared
sudo chown nobody:nogroup /data/shared
sudo chmod 755 /data/shared
```

### 1.3 Konfigurasi Ekspor
```bash
sudo nano /etc/exports
```
Tambahkan:
```
/data/shared 192.168.1.0/24(rw,sync,no_subtree_check)
```

### 1.4 Terapkan Konfigurasi
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

---

## 💻 STEP 2: Konfigurasi NFS Client

### 2.1 Install Paket NFS Client
```bash
sudo apt update
sudo apt install nfs-common -y
```

### 2.2 Buat Mount Point
```bash
sudo mkdir -p /mnt/shared
```

### 2.3 Mount Manual
```bash
sudo mount 192.168.1.10:/data/shared /mnt/shared
```

### 2.4 Verifikasi
```bash
ls -l /mnt/shared
sudo touch /mnt/shared/testfile
```

---

## 🔄 STEP 3: Mount Otomatis (Opsional)

### 3.1 Edit File fstab
```bash
sudo nano /etc/fstab
```
Tambahkan:
```
192.168.1.10:/data/shared  /mnt/shared  nfs  defaults  0  0
```

### 3.2 Test Konfigurasi
```bash
sudo mount -a
```

---

## ⚙️ STEP 4: Opsi Konfigurasi NFS

| Opsi               | Deskripsi                                                                 |
|--------------------|---------------------------------------------------------------------------|
| `rw`               | Read-write access                                                        |
| `ro`               | Read-only access                                                         |
| `sync`             | Write changes synchronously                                              |
| `async`            | Write changes asynchronously                                             |
| `no_subtree_check` | Meningkatkan performa dengan menonaktifkan subtree checking             |
| `no_root_squash`   | Mempertahankan privilege root (hati-hati, risiko keamanan)               |

---

## 🔐 STEP 5: Keamanan NFS

1. Selalu gunakan subnet spesifik di `/etc/exports`
2. Pertimbangkan menggunakan VPN untuk koneksi remote
3. Batasi akses dengan firewall:
   ```bash
   sudo ufw allow from 192.168.1.0/24 to any port nfs
   ```
4. Monitor akses dengan:
   ```bash
   sudo nfsstat -s
   ```

---

## 🗑️ STEP 6: Uninstall NFS

### Di Server:
```bash
sudo systemctl stop nfs-kernel-server
sudo apt remove --purge nfs-kernel-server -y
```

### Di Client:
```bash
sudo umount /mnt/shared
sudo apt remove --purge nfs-common -y
```

---

## 🆘 Troubleshooting

- **Gagal mount**: Cek koneksi jaringan dan konfigurasi `/etc/exports`
- **Permission denied**: Verifikasi ownership folder dan opsi export
- **Slow performance**: Pertimbangkan opsi `async` untuk workload tertentu
- **Port blocked**: Pastikan port 2049 terbuka di firewall

---

## 📚 Referensi
- [Ubuntu NFS Documentation](https://ubuntu.com/server/docs/service-nfs)
- [NFS Best Practices](https://help.ubuntu.com/community/NFSv4Howto)
- [NFS Security Guide](https://linux.die.net/man/5/exports)
