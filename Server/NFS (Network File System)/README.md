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

### 1.5 Cek Daftar Export
```bash
showmount -e localhost
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

## 🔄 STEP 3: Mount Otomatis (Otomatisasi di Client)

Agar mount NFS tidak perlu dilakukan manual setiap kali reboot, konfigurasi otomatisasi dapat dilakukan dengan **dua pendekatan**: melalui **fstab** (sederhana) atau **systemd mount unit** (lebih andal).

---

### 3.1 Opsi A – Konfigurasi via fstab

Metode ini adalah cara klasik dan cepat. Namun, tanpa opsi tambahan, mount bisa gagal jika NFS server belum siap ketika client booting.

Edit file `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Tambahkan baris berikut:

```
192.168.1.10:/data/shared  /mnt/shared  nfs  defaults,_netdev,nofail,x-systemd.automount  0  0
```

**Penjelasan opsi penting**:

* `nofail` → mencegah boot gagal walau mount error.
* `_netdev` → mount dilakukan setelah jaringan aktif.
* `x-systemd.automount` → mount baru dilakukan ketika path pertama kali diakses (on-demand).

Uji coba konfigurasi:

```bash
sudo mount -a
ls /mnt/shared
```

---

### 3.2 Opsi B – Konfigurasi via systemd (Best Practice)

Untuk deployment **production** atau aplikasi kritis, pendekatan dengan `systemd mount unit` lebih disarankan. Dengan ini, mount dikelola seperti service sehingga dapat dipantau, dikontrol, dan diatur dependensinya.

1. Buat file unit:

   ```bash
   sudo nano /etc/systemd/system/mnt-shared.mount
   ```

   Isi dengan:

   ```ini
   [Unit]
   Description=Mount NFS Share
   Requires=network-online.target
   After=network-online.target remote-fs-pre.target
   Before=nginx.service apache2.service

   [Mount]
   What=192.168.1.10:/data/shared
   Where=/mnt/shared
   Type=nfs
   Options=_netdev,hard,timeo=600,retrans=3

   [Install]
   WantedBy=multi-user.target
   ```

2. Aktifkan unit:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable mnt-shared.mount
   sudo systemctl start mnt-shared.mount
   ```

3. Verifikasi status:

   ```bash
   systemctl status mnt-shared.mount
   mount | grep /mnt/shared
   ```

Jika mount ini vital untuk layanan seperti `nginx` atau `php-fpm`, tambahkan dependency agar service tersebut menunggu mount selesai:

```ini
Before=nginx.service php8.3-fpm.service
```

---

## ⚙️ STEP 4: Opsi Konfigurasi NFS

| Opsi                  | Deskripsi                                                                             |
| --------------------- | ------------------------------------------------------------------------------------- |
| `rw`                  | Read-write access (default: baca & tulis)                                             |
| `ro`                  | Read-only access (hanya baca, aman untuk share publik)                                |
| `sync`                | Write dilakukan secara sinkron (lebih aman, tapi lebih lambat)                        |
| `async`               | Write dilakukan secara asinkron (lebih cepat, tapi risiko data corruption jika crash) |
| `no_subtree_check`    | Menonaktifkan subtree checking → meningkatkan performa pada direktori besar           |
| `subtree_check`       | Default, mengecek permission subtree (lebih aman, sedikit lebih lambat)               |
| `no_root_squash`      | User root di client tetap jadi root di server (⚠️ risiko tinggi)                      |
| `root_squash`         | User root di client dipetakan ke `nobody` di server (default, lebih aman)             |
| `_netdev`             | Tandai filesystem sebagai berbasis jaringan, mount hanya setelah network aktif        |
| `nofail`              | Jika gagal mount, sistem tetap boot normal tanpa error                                |
| `x-systemd.automount` | Mount otomatis hanya saat direktori pertama kali diakses                              |
| `hard`                | Client akan retry terus jika server NFS tidak merespons (default, lebih aman)         |
| `soft`                | Client akan berhenti mencoba dan kembalikan error jika NFS server tidak merespons     |
| `timeo=<n>`           | Timeout RPC dalam decisecond (contoh `timeo=600` → 60 detik)                          |
| `retrans=<n>`         | Jumlah percobaan ulang RPC sebelum error (default biasanya 3)                         |

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
