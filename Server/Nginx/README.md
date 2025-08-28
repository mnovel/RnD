# 🚀 Panduan Instalasi Web Server: Nginx + PHP + Ekstensi di Ubuntu 24.04

Dokumentasi ini mencakup langkah-langkah instalasi dan konfigurasi Nginx, PHP, serta ekstensi umum yang sering digunakan pada web server berbasis Linux Ubuntu.

---

## 🧰 Prasyarat

- Ubuntu 24.04 LTS
- Akses user dengan hak `sudo`
- Koneksi internet aktif

---

## 🌐 STEP 1: Instalasi Nginx

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

### 🔓 1.1 Buka akses firewall (opsional jika UFW aktif)

```bash
sudo ufw allow 'Nginx Full'
```

---

## 🐘 STEP 2: Instalasi PHP dan Ekstensi Umum

```bash
sudo apt install php php-cli php-fpm php-common php-mysql php-pgsql php-sqlite3 \
php-curl php-gd php-mbstring php-xml php-bcmath php-zip php-soap php-intl php-readline \
php-opcache php-tokenizer php-ctype php-dom php-fileinfo php-exif php-pdo -y
```

> 💡 Secara default, Ubuntu 24.04 menggunakan PHP 8.3.

---

## 📦 STEP 3: Daftar Ekstensi PHP Umum

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

## 🔧 STEP 4: Konfigurasi Nginx agar Bisa Jalankan PHP

### 4.1 Edit konfigurasi default Nginx:

```bash
sudo nano /etc/nginx/sites-available/default
```

### 4.2 Contoh konfigurasi `server` block:

Config NGINX Untuk redirect HTTP ke HTTPS

```nginx
# ======================
# REDIRECT HTTP → HTTPS
# ======================
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    return 301 https://$host$request_uri;
}
```

Config NGINX Untuk PHP

```nginx
server {
    # ======================
    # BASIC SERVER CONFIGURATION
    # ======================
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    # Untuk SSL
    # listen 443 ssl http2;
    # listen [::]:443 ssl http2;
    # server_name example.com www.example.com;

    # Real IP configuration
    set_real_ip_from 10.10.1.0/24;
    real_ip_header X-Forwarded-For;
    real_ip_recursive on;

    # ======================
    # SSL CERTIFICATE
    # ======================
    # ssl_certificate     /etc/nginx/ssl/server.fullchain.crt;
    # ssl_certificate_key /etc/nginx/ssl/server.key;
    # ssl_trusted_certificate /etc/nginx/ssl/server.trusted.crt;

    # ======================
    # SSL SETTINGS
    # ======================
    # ssl_protocols TLSv1.2 TLSv1.3;
    # ssl_prefer_server_ciphers on;
    # ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    # ssl_session_timeout 1d;
    # ssl_session_cache shared:SSL:50m;
    # ssl_session_tickets off;
    # ssl_stapling on;
    # ssl_stapling_verify on;
    # resolver 1.1.1.1 1.0.0.1 valid=300s;
    # resolver_timeout 5s;
    
    # ======================
    # SECURITY HEADERS
    # ======================
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), fullscreen=(self)" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header Content-Security-Policy "default-src 'self' data: blob: https: http:; script-src 'self' 'unsafe-inline' https: http:; style-src 'self' 'unsafe-inline' https: http:; img-src 'self' data: https: http:; font-src 'self' data: https: http:; connect-src 'self' https: http:; frame-ancestors 'self';" always;

    # ======================
    # FRAMEWORK-SPECIFIC CONFIG
    # ======================
    # Untuk Laravel
    root /var/www/html/public;
    index index.php index.html;
    
    # Untuk CodeIgniter
    # root /var/www/html;
    # index index.php index.html;

    # ======================
    # ROUTING RULES
    # ======================
    location / {
        # Untuk Laravel:
        try_files $uri $uri/ /index.php$is_args$args;
        
        # Untuk CodeIgniter:
        # try_files $uri $uri/ /index.php?$query_string;
    }

    # ======================
    # PHP HANDLER
    # ======================
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        
        # Untuk Laravel:
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        
        # Untuk CodeIgniter:
        # fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        # fastcgi_param PATH_INFO $fastcgi_path_info;
        
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;

        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
    }

    # ======================
    # SECURITY RESTRICTIONS
    # ======================
    # Blokir file/direktori sensitif framework
    location ~* ^/(\.env|\.git|storage/logs|bootstrap/cache|config/|database/|application|system|tests|composer\.(json|lock)) {
        deny all;
        return 403;
    }

    # Blokir ekstensi file berbahaya
    location ~* \.(engine|inc|info|install|module|profile|test|po|sh|.*sql|theme|tpl(\.php)?|xtmpl|sw[op]|xmlrpc)$ {
        deny all;
        return 403;
    }

    # Blokir file hidden kecuali .well-known
    location ~ /\.(?!well-known) {
        deny all;
        access_log off;
        log_not_found off;
    }

    # ======================
    # STATIC FILES CACHING
    # ======================
    location ~* \.(?:ico|css|js|gif|jpe?g|png|svg|webp|woff2?|ttf|eot)$ {
        expires 365d;
        add_header Cache-Control "public, immutable";
        access_log off;
        try_files $uri =404;
    }

    # ======================
    # MISC CONFIG
    # ======================
    client_max_body_size 100M;
    keepalive_timeout 15;
    
    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }
    
    location = /robots.txt {
        log_not_found off;
        access_log off;
    }
}
```

Config NGINX Untuk NodeJs

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    # Real IP Configuration
    set_real_ip_from 10.10.1.0/24;
    real_ip_header X-Forwarded-For;
    real_ip_recursive on;

    # Basic Directives
    root /var/www/myapp/build;
    index index.html;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), fullscreen=(self)" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'self'; base-uri 'self';" always;

    # Rewrite untuk SPA (Single Page Application)
    location / {
        try_files $uri /index.html;
    }

    # Caching untuk static files
    location ~* \.(?:ico|css|js|gif|jpe?g|png|woff2?|ttf|svg|eot|webp)$ {
        expires 365d;
        add_header Cache-Control "public, immutable";
        access_log off;
        try_files $uri =404;
    }

    # Blok akses file tersembunyi dan sensitif
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}

```

> ⚠️ Ganti `php8.3-fpm.sock` sesuai versi PHP yang terinstal.

### 4.3 Tambah blacklist Domain Referer (Otomatis)

Untuk memblokir spam referer secara otomatis, kita akan menggunakan **map** dari Nginx yang di-generate lewat script cron.

#### 4.3.1 Buat Script Generator Blacklist

```bash
sudo nano /usr/local/bin/update-nginx-blacklist.sh
```

Isi script:

```bash
#!/bin/bash
set -e

BLACKLIST_CONF="/etc/nginx/blacklist.conf"
TMP_FILE=$(mktemp)

curl -s https://raw.githubusercontent.com/mnovel/RnD/refs/heads/main/Server/Nginx/blacklist.txt \
| sed 's/^/~/' \
| awk '{print $1" 1;"}' \
| sed '1i map $http_referer $bad_referer { hostnames; default 0;' \
| sed '$a }' > "$TMP_FILE"

if [ ! -f "$BLACKLIST_CONF" ]; then
    echo "[INFO] $BLACKLIST_CONF tidak ditemukan, membuat baru dengan konfigurasi default..."
    cat > "$BLACKLIST_CONF" <<EOF
map \$http_referer \$bad_referer {
    hostnames;
    default 0;
}
EOF
fi

if nginx -t -q; then
    mv "$TMP_FILE" "$BLACKLIST_CONF"
    nginx -t && systemctl reload nginx
    echo "[OK] Blacklist updated & nginx reloaded."
else
    echo "[ERROR] Invalid nginx config. Blacklist not updated."
    rm -f "$TMP_FILE"
    exit 1
fi
```

Kasih permission:

```bash
sudo chmod +x /usr/local/bin/update-nginx-blacklist.sh
```


#### 4.3.2 Integrasi dengan Nginx

Edit konfigurasi server block (`/etc/nginx/sites-available/default`):

```nginx
include /etc/nginx/blacklist.conf;

if ($bad_referer) {
    return 403;
}
```

#### 4.3.3 Tambah Cron Job

Agar selalu update otomatis, jalankan script setiap hari jam 02:00 pagi:

```bash
sudo crontab -e
```

Isi dengan:

```
0 2 * * * /usr/local/bin/update-nginx-blacklist.sh >/dev/null 2>&1
```

---

## 🔁 STEP 5: Restart Service

```bash
sudo systemctl reload nginx
sudo systemctl restart php8.3-fpm
```

---

## 🧪 STEP 6: Pengujian

### 6.1 Buat file uji `info.php`

```bash
echo "<?php phpinfo();" | sudo tee /var/www/html/info.php
```

### 6.2 Akses via browser:

```
http://<ip-server>/info.php
```

---

## 🗑️ STEP 7: Uninstall PHP (Jika Perlu)

```bash
sudo apt remove --purge php* -y
sudo apt autoremove -y
```

---

## 📌 STEP 8: Catatan Tambahan

- Untuk versi PHP lainnya, gunakan PPA:

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.2 php8.2-fpm ...
```
