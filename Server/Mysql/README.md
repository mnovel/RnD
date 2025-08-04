# 🗄️ Panduan Instalasi MySQL di Ubuntu 24.04

Dokumentasi lengkap untuk menginstal dan mengamankan MySQL server di Ubuntu 24.04 (Noble Numbat).

---

## 🧰 Prasyarat
- Ubuntu 24.04 LTS
- Akses user dengan hak `sudo`
- Koneksi internet aktif
- Disarankan minimal 2GB RAM untuk server produksi

---

## 🔄 STEP 1: Update Sistem
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 📦 STEP 2: Instalasi MySQL Server
```bash
sudo apt install mysql-server -y
```

---

## ✅ STEP 3: Verifikasi Instalasi
```bash
sudo systemctl status mysql
```
Output yang diharapkan:
```
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

---

## 🔐 STEP 4: Konfigurasi Keamanan
Jalankan skrip pengamanan:
```bash
sudo mysql_secure_installation
```

Ikuti pengaturan berikut:
- Validasi password: **Yes** (opsional)
- Hapus user anonymous: **Yes**
- Nonaktifkan login root remote: **Yes**
- Hapus database test: **Yes**
- Muat ulang privilege tables: **Yes**

---

## 🔑 STEP 5: Akses MySQL
### 5.1 Login ke MySQL
```bash
sudo mysql -u root -p
```

### 5.2 Contoh buat user dan database baru:
```sql
CREATE DATABASE contohdb;
CREATE USER 'contohuser'@'localhost' IDENTIFIED BY 'PasswordKuat123!';
GRANT ALL PRIVILEGES ON contohdb.* TO 'contohuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 🌐 STEP 6: Konfigurasi Akses Remote (Opsional)
### 6.1 Edit file konfigurasi:
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Ubah:
```ini
bind-address = 127.0.0.1
```
Menjadi:
```ini
bind-address = 0.0.0.0
```

### 6.2 Buat user remote:
```sql
CREATE USER 'remoteuser'@'%' IDENTIFIED BY 'PasswordRemote456!';
GRANT ALL PRIVILEGES ON *.* TO 'remoteuser'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 6.3 Restart MySQL:
```bash
sudo systemctl restart mysql
```

---

## 🛠️ STEP 7: Perintah MySQL Penting
| Perintah | Deskripsi |
|----------|-----------|
| `sudo systemctl start mysql` | Mulai layanan MySQL |
| `sudo systemctl stop mysql` | Hentikan layanan MySQL |
| `sudo systemctl restart mysql` | Restart layanan MySQL |
| `mysql -u [user] -p` | Login ke MySQL CLI |
| `SHOW DATABASES;` | Tampilkan semua database |
| `CREATE DATABASE nama_db;` | Buat database baru |

---

## ⚠️ STEP 8: Keamanan Tambahan
1. Selalu gunakan password yang kuat
2. Batasi akses remote hanya dari IP tertentu
3. Pertimbangkan menggunakan SSH tunneling
4. Update MySQL secara berkala

---

## 🗑️ STEP 9: Uninstall MySQL
```bash
sudo apt remove --purge mysql-server mysql-client mysql-common -y
sudo apt autoremove -y
sudo apt autoclean
sudo rm -rf /etc/mysql /var/lib/mysql
```

---

## 🆘 Troubleshooting
- **Gagal start**: Cek log dengan `sudo journalctl -u mysql.service`
- **Login gagal**: Verifikasi password dengan `sudo mysql -u root`
- **Port tertutup**: Buka port 3306 di firewall `sudo ufw allow 3306`

---

## 📚 Referensi
- [Dokumentasi Resmi MySQL](https://dev.mysql.com/doc/)
- [Panduan Ubuntu MySQL](https://ubuntu.com/server/docs/databases-mysql)
- [Best Practices Keamanan MySQL](https://dev.mysql.com/doc/refman/8.0/en/security.html)
