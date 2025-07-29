# 🔁 Rsync Documentation

`rsync` adalah alat baris perintah (CLI) yang digunakan untuk menyinkronkan dan mentransfer file atau direktori antara dua lokasi, baik secara lokal maupun melalui jaringan menggunakan protokol SSH.

---

## ✨ Fitur Unggulan

* Transfer data **cepat dan efisien**
* Hanya mengirimkan **perubahan (delta)** file
* Mendukung sinkronisasi **lokal dan remote**
* Dapat mempertahankan **izin file, timestamp, dan symbolic links**
* Kompatibel dengan **SSH untuk keamanan**

---

## 📦 Instalasi

### Ubuntu/Debian:

```bash
sudo apt update && sudo apt install rsync
```

### CentOS/RHEL:

```bash
sudo yum install rsync
```

### macOS:

```bash
brew install rsync
```

---

## 🧪 Sintaks Umum

```bash
rsync [OPTION]... SRC [USER@]HOST:DEST
```

atau

```bash
rsync [OPTION]... [USER@]HOST:SRC DEST
```

---

## 🔍 Contoh Penggunaan

### 1. 🔁 Sinkronisasi Lokal (Folder ke Folder)

```bash
rsync -avh /source/folder/ /destination/folder/
```

### 2. ☁️ Kirim File ke Remote Server

```bash
rsync -avh /local/folder/ user@remote_ip:/remote/folder/
```

### 3. 📅 Ambil File dari Remote Server

```bash
rsync -avh user@remote_ip:/remote/folder/ /local/folder/
```

### 4. 🔐 Gunakan Port SSH Khusus

```bash
rsync -avh -e "ssh -p 2222" /local/folder/ user@remote_ip:/remote/folder/
```

---

## ⚙️ Opsi yang Sering Digunakan

| Opsi                  | Keterangan                                    |
| --------------------- | --------------------------------------------- |
| `-a`                  | Archive: sync recursive, izin, waktu, symlink |
| `-v`                  | Verbose: tampilkan proses                     |
| `-h`                  | Human-readable: ukuran file mudah dibaca      |
| `-z`                  | Kompresi data selama transfer                 |
| `--delete`            | Hapus file di tujuan yang tidak ada di sumber |
| `--progress`          | Tampilkan progress setiap file                |
| `--exclude='pattern'` | Mengecualikan file atau folder tertentu       |

---

## 🕒 Automasi via Cron

### Edit crontab:

```bash
crontab -e
```

### Contoh: Backup harian pukul 1 pagi

```bash
0 1 * * * rsync -avh /source/folder/ /backup/folder/ >> /var/log/rsync.log 2>&1
```

---

## 📌 Tips Penting

* Akhiri path sumber dengan `/` untuk **menyalin isi folder**, tanpa `/` untuk **menyalin folder itu sendiri**.
* Gunakan SSH key agar proses otomatis tidak memerlukan password.
* Selalu uji coba `rsync` tanpa `--delete` terlebih dahulu untuk menghindari kehilangan data.

---

## 🔐 SSH Key Setup (Opsional)

### Di client (server yang menarik data):

```bash
ssh-keygen -t rsa
ssh-copy-id -p 234 user@remote_ip
```

Ini memungkinkan `rsync` dijalankan tanpa perlu memasukkan password SSH.

---

## 🧠 Contoh Nyata

### Backup Website ke NAS:

```bash
rsync -avzh /var/www/html/ /mnt/nas/backup/html/
```

### Sinkronisasi Server Produksi ke Server Backup:

```bash
rsync -avh -e "ssh -p 2222" user@prod-server:/data/ /data/backup/
```

---

## 📚 Referensi

* [Rsync Manual](https://man7.org/linux/man-pages/man1/rsync.1.html)
* [Explained: Rsync Basics](https://explainshell.com/explain?cmd=rsync+-avh)

---

## 🛡️ Disclaimer

Gunakan `--delete` dengan hati-hati! Selalu lakukan dry-run terlebih dahulu:

```bash
rsync -avhn --delete source/ destination/
```
