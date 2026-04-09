# 🐘 Panduan Instalasi PostgreSQL di Ubuntu 24.04

Dokumentasi lengkap untuk menginstal dan mengamankan PostgreSQL server di Ubuntu 24.04 (Noble Numbat).

---

## 🧰 Prasyarat

* Ubuntu 24.04 LTS
* Akses user dengan hak `sudo`
* Koneksi internet aktif
* Disarankan minimal 2GB RAM untuk server produksi

---

## 🔄 STEP 1: Update Sistem

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 📦 STEP 2: Instalasi PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

---

## ✅ STEP 3: Verifikasi Instalasi

```bash
sudo systemctl status postgresql
```

Output yang diharapkan:

```
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled)
     Active: active (running) since ...
```

---

## 🔐 STEP 4: Konfigurasi User PostgreSQL

PostgreSQL menggunakan user sistem `postgres`.

### 4.1 Masuk ke shell PostgreSQL:

```bash
sudo -i -u postgres
psql
```

### 4.2 Set password untuk user postgres:

```sql
ALTER USER postgres PASSWORD 'PasswordKuat123!';
```

---

## 🔑 STEP 5: Membuat Database dan User

```sql
CREATE DATABASE contohdb;
CREATE USER contohuser WITH ENCRYPTED PASSWORD 'PasswordKuat123!';
ALTER ROLE contohuser SET client_encoding TO 'utf8';
ALTER ROLE contohuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE contohuser SET timezone TO 'Asia/Jakarta';

GRANT ALL PRIVILEGES ON DATABASE contohdb TO contohuser;
```

Keluar:

```bash
\q
exit
```

---

## 🌐 STEP 6: Konfigurasi Akses Remote (Opsional)

### 6.1 Edit konfigurasi PostgreSQL:

```bash
sudo nano /etc/postgresql/*/main/postgresql.conf
```

Cari dan ubah:

```ini
#listen_addresses = 'localhost'
```

Menjadi:

```ini
listen_addresses = '*'
```

---

### 6.2 Konfigurasi akses client:

```bash
sudo nano /etc/postgresql/*/main/pg_hba.conf
```

Tambahkan:

```
host    all             all             0.0.0.0/0               md5
```

⚠️ **Rekomendasi:** Batasi ke IP tertentu, contoh:

```
host    all             all             192.168.1.0/24          md5
```

---

### 6.3 Restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

---

## 🛠️ STEP 7: Perintah PostgreSQL Penting

| Perintah                            | Deskripsi                   |
| ----------------------------------- | --------------------------- |
| `sudo systemctl start postgresql`   | Mulai layanan PostgreSQL    |
| `sudo systemctl stop postgresql`    | Hentikan layanan PostgreSQL |
| `sudo systemctl restart postgresql` | Restart layanan PostgreSQL  |
| `sudo -u postgres psql`             | Login ke PostgreSQL CLI     |
| `\l`                                | List database               |
| `\du`                               | List user/role              |
| `\c nama_db`                        | Koneksi ke database         |
| `CREATE DATABASE nama_db;`          | Buat database               |

---

## ⚠️ STEP 8: Keamanan Tambahan

1. Gunakan password yang kuat
2. Batasi akses di `pg_hba.conf`
3. Gunakan firewall untuk port 5432
4. Pertimbangkan SSL connection
5. Update PostgreSQL secara berkala

---

## 🔥 STEP 9: Konfigurasi Firewall (Opsional)

```bash
sudo ufw allow 5432
```

---

## 🗑️ STEP 10: Uninstall PostgreSQL

```bash
sudo apt remove --purge postgresql postgresql-contrib -y
sudo apt autoremove -y
sudo apt autoclean
sudo rm -rf /etc/postgresql /var/lib/postgresql
```

---

## 🆘 Troubleshooting

* **Gagal start**:

  ```bash
  sudo journalctl -u postgresql
  ```

* **Tidak bisa login**:
  Cek metode auth di `pg_hba.conf`

* **Port tidak terbuka**:

  ```bash
  sudo ss -tunlp | grep 5432
  ```

---

## 📚 Referensi

* [https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)
* [https://ubuntu.com/server/docs/databases-postgresql](https://ubuntu.com/server/docs/databases-postgresql)
* [https://www.postgresql.org/docs/current/security.html](https://www.postgresql.org/docs/current/security.html)
