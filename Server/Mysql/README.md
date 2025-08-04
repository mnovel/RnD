# 📦 Instalasi MySQL di Ubuntu Server 24.04

Dokumentasi ini menjelaskan langkah-langkah untuk menginstal dan mengamankan MySQL di Ubuntu Server 24.04 LTS.

## 🛠️ Prasyarat

- Ubuntu Server 24.04 LTS
- Akses root atau sudo

---

## 🔧 Langkah Instalasi

### 1. Perbarui Sistem

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Instal MySQL Server

```bash
sudo apt install mysql-server -y
```

### 3. Cek Status Layanan MySQL

```bash
sudo systemctl status mysql
```

Jika aktif, output akan menunjukkan status: `active (running)`.

---

## 🔐 Konfigurasi Keamanan

### 4. Jalankan `mysql_secure_installation`

```bash
sudo mysql_secure_installation
```

Ikuti prompt berikut:
- Atur validasi kata sandi: **Yes** (opsional)
- Masukkan root password: **Isi jika diminta**
- Hapus pengguna anonim: **Yes**
- Nonaktifkan login root dari remote: **Yes**
- Hapus database test: **Yes**
- Muat ulang tabel privilege: **Yes**

---

## 🔑 Akses MySQL

### 5. Masuk ke MySQL CLI

```bash
sudo mysql
```

### 6. (Opsional) Buat User Baru dan Database

```sql
CREATE DATABASE namadb;
CREATE USER 'namauser'@'localhost' IDENTIFIED BY 'passwordku';
GRANT ALL PRIVILEGES ON namadb.* TO 'namauser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 🔍 Verifikasi Instalasi

### 7. Cek Versi MySQL

```bash
mysql --version
```

### 8. Tes Koneksi

```bash
mysql -u root -p
```

---

## 🧱 Konfigurasi Tambahan (Opsional)

### Mengizinkan Akses Remote ke MySQL

1. Edit file konfigurasi:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

2. Ubah `bind-address` dari:

```ini
bind-address = 127.0.0.1
```

Menjadi:

```ini
bind-address = 0.0.0.0
```

3. Restart MySQL:

```bash
sudo systemctl restart mysql
```

4. Tambahkan user remote di MySQL:

```sql
CREATE USER 'remoteuser'@'%' IDENTIFIED BY 'passwordku';
GRANT ALL PRIVILEGES ON *.* TO 'remoteuser'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

---

## 🧼 Uninstall (Jika Diperlukan)

```bash
sudo apt remove --purge mysql-server mysql-client mysql-common -y
sudo apt autoremove -y
sudo apt autoclean
sudo rm -rf /etc/mysql /var/lib/mysql
```

---

## 📚 Referensi

- [MySQL Official Documentation](https://dev.mysql.com/doc/)
- [Ubuntu Community Help Wiki](https://help.ubuntu.com/community/MySQL)

---

## 🧑‍💻 Author

**Nama**: Novel  
**Role**: DevOps | SysAdmin | Cybersecurity  
**Organization**: RSUD Dr. R Soedarsono
