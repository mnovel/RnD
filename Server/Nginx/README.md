# 🚀 Panduan Instalasi Web Server: Nginx + PHP + Ekstensi di Ubuntu 24.04

Dokumentasi ini mencakup langkah-langkah instalasi dan konfigurasi Nginx, PHP, serta ekstensi umum yang sering digunakan pada web server berbasis Linux Ubuntu.

---

## 🧰 Prasyarat

* Ubuntu 24.04 LTS
* Akses user dengan hak `sudo`
* Koneksi internet aktif

---

## 🌐 Langkah 1: Instalasi Nginx

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

### 🔓 Buka akses firewall (opsional jika UFW aktif)

```bash
sudo ufw allow 'Nginx Full'
```

---

## 🐘 Langkah 2: Instalasi PHP dan Ekstensi Umum

```bash
sudo apt install php php-cli php-fpm php-common php-mysql php-pgsql php-sqlite3 \
php-curl php-gd php-mbstring php-xml php-bcmath php-zip php-soap php-intl php-readline \
php-opcache php-tokenizer php-ctype php-dom php-fileinfo php-exif php-pdo -y
```

> 💡 Secara default, Ubuntu 24.04 menggunakan PHP 8.3.

### 📦 Daftar Ekstensi PHP Umum

| Ekstensi      | Fungsi                                         |
| ------------- | ---------------------------------------------- |
| php-cli       | Menjalankan PHP via command-line               |
| php-fpm       | PHP FastCGI Process Manager                    |
| php-mysql     | Koneksi ke database MySQL/MariaDB              |
| php-pgsql     | Koneksi ke PostgreSQL                          |
| php-sqlite3   | Dukungan database SQLite3                      |
| php-curl      | Akses API eksternal melalui HTTP               |
| php-gd        | Pemrosesan dan manipulasi gambar               |
| php-mbstring  | String multibyte (penting untuk framework PHP) |
| php-xml       | Parsing dan manipulasi XML                     |
| php-bcmath    | Perhitungan numerik presisi tinggi             |
| php-zip       | Kompresi dan dekompresi file zip               |
| php-soap      | SOAP web service                               |
| php-intl      | Internasionalisasi (i18n)                      |
| php-readline  | Fitur interaktif CLI                           |
| php-opcache   | Caching bytecode PHP                           |
| php-tokenizer | Parsing token PHP                              |
| php-ctype     | Validasi karakter                              |
| php-dom       | DOM XML support                                |
| php-fileinfo  | Identifikasi tipe file                         |
| php-exif      | Metadata gambar (Exif)                         |
| php-pdo       | Abstraksi koneksi database                     |

---

## 🔧 Langkah 3: Konfigurasi Nginx agar Bisa Jalankan PHP

Edit konfigurasi default Nginx:

```bash
sudo nano /etc/nginx/sites-available/default
```

### Contoh konfigurasi `server` block:

```nginx
server {
    listen 80 default_server;
    root /var/www/html;
    index index.php index.html index.htm;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

> ⚠️ Ganti `php8.3-fpm.sock` sesuai versi PHP yang terinstal.

---

## 🔁 Restart Service

```bash
sudo systemctl reload nginx
sudo systemctl restart php8.3-fpm
```

---

## 🧪 Langkah 4: Pengujian

### Buat file uji `info.php`

```bash
echo "<?php phpinfo();" | sudo tee /var/www/html/info.php
```

### Akses via browser:

```
http://<ip-server>/info.php
```

Jika muncul tampilan informasi PHP, berarti server telah terkonfigurasi dengan benar.

---

## 🗑️ Uninstall PHP (jika perlu)

```bash
sudo apt remove --purge php* -y
sudo apt autoremove -y
```

---

## 📌 Catatan Tambahan

* Untuk versi PHP lainnya, gunakan PPA:

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.2 php8.2-fpm ...
```
