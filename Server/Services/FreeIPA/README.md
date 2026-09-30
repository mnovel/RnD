# 📑 Panduan Instalasi & Konfigurasi FreeIPA

## FreeIPA (Server Rocky Linux 9 & Client Ubuntu 24)

Dokumentasi ini menjelaskan setup **FreeIPA** sebagai **Identity Management / Central Authentication** server untuk **Linux/Ubuntu Client**, termasuk integrasi **SSSD, Kerberos, dan automount home directory**.

---

## 🧰 Prasyarat

**Server (Rocky Linux 9)**

* Rocky Linux 9 minimal install
* Akses `sudo` / root
* Hostname lengkap (FQDN) dan DNS berfungsi
* Sinkronisasi waktu (NTP/chrony)
* Port terbuka di firewall:

  * TCP/UDP 88 (Kerberos)
  * TCP/UDP 389 (LDAP)
  * TCP 636 (LDAPS)
  * TCP 443 (HTTP/HTTPS Web UI)
  * TCP 7389 (IPA replication, optional)

**Client (Ubuntu 24)**

* Ubuntu 24 LTS
* Akses `sudo`
* Koneksi ke server FreeIPA
* Paket `freeipa-client` akan dipasang
* Waktu sinkron dengan server (NTP)
* Port firewall client terbuka (lihat STEP 3.1)

---

## 🏗️ Arsitektur

```text
          +-------------------------+
          |   FreeIPA Server        |
          | rocky9-ipa.rsud.internal|
          | 10.10.1.10             |
          +-------------------------+
                     ^
                     | TCP/UDP 88, 389, 443
    -------------------------------------------
    |                     |                   |
+-----------+       +-----------+       +-----------+
| Client A  |       | Client B  |  ...  | Client N  |
| Ubuntu 24 |       | Ubuntu 24 |       | Ubuntu 24 |
+-----------+       +-----------+       +-----------+
```

---

## 🌐 STEP 1: Instalasi FreeIPA Server (Rocky 9)

### 1.1 Update sistem

```bash
sudo dnf update -y
sudo dnf install epel-release -y
```

### 1.2 Instal paket FreeIPA

```bash
sudo dnf install ipa-server ipa-server-dns -y
```

> Jika ingin sekaligus sebagai DNS server, tambahkan `ipa-server-dns`.

### 1.3 Setup FreeIPA Server

```bash
sudo ipa-server-install
```

Ikuti prompt:

* Nama domain: `rsud.internal`
* Nama realm: `RSUD.INTERNAL`
* IPA admin password
* Konfirmasi DNS (opsional jika pakai internal DNS)
* Pilih otomatis konfigurasi Kerberos/LDAP

---

### 1.4 Aktifkan dan cek service

```bash
sudo systemctl enable --now ipa
sudo systemctl status ipa
```

---

## 🖥️ STEP 2: Konfigurasi Firewall (Server)

```bash
sudo firewall-cmd --add-service=freeipa-ldap --permanent
sudo firewall-cmd --add-service=freeipa-ldaps --permanent
sudo firewall-cmd --add-service=kerberos --permanent
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --reload
```

---

## 🌐 STEP 3: Instalasi FreeIPA Client (Ubuntu 24)

### 3.1 Update sistem & buka port firewall client

> **Penting:** Langkah ini sering dilupakan. Server FreeIPA punya port terbuka, tapi **client juga harus bisa mengakses** port tersebut keluar. Jika memakai `ufw` (default di Ubuntu server), port harus dibuka eksplisit — kalau tidak, `ipa-client-install` akan gagal di tahap `kinit` dengan error `Cannot read password while getting initial credentials`.

```bash
sudo apt update && sudo apt upgrade -y

# Buka port yang dibutuhkan untuk enrollment
sudo ufw allow 80/tcp
sudo ufw allow 88/tcp
sudo ufw allow 88/udp
sudo ufw allow 389/tcp
sudo ufw allow 464/tcp
sudo ufw allow 464/udp
sudo ufw allow 123/udp      # jika pakai NTP

# Cek status
sudo ufw status verbose
```

> Jika client memakai `nftables` / `iptables` langsung, sesuaikan sintaksnya.

### 3.2 Sinkronisasi waktu (NTP)

Kerberos sangat sensitif terhadap **clock skew** — perbedaan waktu antara client dan server lebih dari **5 menit** akan menyebabkan `kinit` gagal.

```bash
sudo apt install chrony -y
sudo systemctl enable --now chrony

# Arahkan ke server IPA sebagai sumber waktu
sudo tee /etc/chrony/chrony.conf >/dev/null <<'EOF'
server rocky9-ipa.rsud.internal iburst
driftfile /var/lib/chrony/chrony.drift
makestep 1.0 3
rtcsync
EOF

sudo systemctl restart chrony

# Paksa sinkronisasi langsung
sudo chronyc makestep
sudo chronyc sources
```

### 3.3 Verifikasi DNS & Hostname

```bash
# Hostname harus FQDN
hostname -f
# Contoh output: sehat.rsud.internal

# Server IPA harus resolve
getent hosts rocky9-ipa.rsud.internal

# DNS resolver harus mengarah ke server IPA (atau DNS yang forward ke IPA)
cat /etc/resolv.conf
```

Jika hostname belum FQDN:

```bash
sudo hostnamectl set-hostname sehat.rsud.internal
```

### 3.4 Install FreeIPA client

```bash
sudo apt install freeipa-client -y
```

### 3.5 Join Client ke FreeIPA Server

Perintah `ipa-client-install` dengan parameter lengkap (jangan hanya `--mkhomedir --enable-dns-updates`):

```bash
sudo ipa-client-install \
    --mkhomedir \
    --enable-dns-updates \
    --domain=rsud.internal \
    --server=rocky9-ipa.rsud.internal \
    --realm=RSUD.INTERNAL \
    --principal=admin \
    --hostname=$(hostname -f) \
    --ntp-server=rocky9-ipa.rsud.internal \
    --force-join
```

> * `--mkhomedir` membuat **home directory otomatis** saat user login.
> * `--ntp-server` memastikan NTP client diarahkan ke server IPA.
> * `--force-join` berguna jika ada sisa state dari percobaan sebelumnya.
> * Ikuti prompt password admin FreeIPA.

Jika enrollment gagal, lihat **STEP 7** untuk troubleshooting.

---

## 🖥️ STEP 4: Konfigurasi SSSD (Client)

### 4.1 Edit `/etc/sssd/sssd.conf`

> **Perhatian:** Baris `services` **harus memuat `sudo`** agar kebijakan sudo dari FreeIPA bisa dibaca. Tanpa ini, `sudo` rule dari server **tidak akan pernah berfungsi**, meskipun `sudo_provider = ipa` sudah diset.

Cek `/etc/sssd/sssd.conf`:

```ini
[domain/rsud.internal]
id_provider = ipa
ipa_server = _srv_, rocky9-ipa.rsud.internal
ipa_domain = rsud.internal
ipa_hostname = sehat.rsud.internal
auth_provider = ipa
chpass_provider = ipa
access_provider = ipa
cache_credentials = True
override_homedir = /home/%u
default_shell = /bin/bash
sudo_provider = ipa
ldap_tls_cacert = /etc/ipa/ca.crt
dyndns_update = True
dyndns_iface = ens18
krb5_store_password_if_offline = True
entry_cache_timeout = 300
sudo_cache_timeout = 60
hbac_cache_timeout = 60

[sssd]
services = nss, pam, ssh, sudo
config_file_version = 2
domains = rsud.internal

[nss]
homedir_substring = /home

[pam]

[sudo]

[autofs]

[ssh]

[pac]

[ifp]

[session_recording]
```

### 4.2 Pastikan `/etc/nsswitch.conf` memuat `sss` untuk sudoers

Tanpa baris ini, `sudo.ws` tidak akan membaca kebijakan sudo dari SSSD/FreeIPA.

```bash
grep sudoers /etc/nsswitch.conf
# Harus menghasilkan:
# sudoers: files sss
```

Jika belum ada, tambahkan manual:

```bash
sudo sed -i 's/^sudoers:.*/sudoers: files sss/' /etc/nsswitch.conf
# Atau jika baris sudoers belum ada sama sekali:
echo "sudoers: files sss" | sudo tee -a /etc/nsswitch.conf
```

### 4.3 ⚠️ PENTING: Cek Implementasi `sudo` (Ubuntu 24.04+ / 26.04)

> **Ini adalah penyebab paling sering kegagalan `sudo` di klien FreeIPA pada Ubuntu versi baru.**
>
> Sejak Ubuntu 24.04, Canonical mulai mengadopsi **`sudo-rs`** (implementasi `sudo` berbasis Rust). Di Ubuntu 26.04, `sudo-rs` menjadi **default**. Masalahnya: **`sudo-rs` belum mendukung pembacaan kebijakan sudo dari SSSD/FreeIPA**. Ia hanya membaca `/etc/sudoers` lokal, sehingga semua sudo rule dari FreeIPA **tidak akan terbaca**.
>
> Gejala: enrollment sukses, login user sukses, tapi `sudo` menolak dengan pesan:
>
> ```
> sudo: Sorry, user novel may not run sudo on <host>.
> ```

Cek implementasi yang aktif:

```bash
sudo -V | head -1
```

* Jika output: `Sudo version 1.9.x` → aman, lanjut ke 4.4.
* Jika output: `sudo-rs ...` → **ganti ke `sudo.ws`**:

```bash
# Cara interaktif
sudo update-alternatives --config sudo
# Pilih opsi yang mengarah ke /usr/bin/sudo.ws

# Atau cara cepat non-interaktif
sudo update-alternatives --set sudo /usr/bin/sudo.ws

# Verifikasi
sudo -V | head -1
# Harus menampilkan: Sudo version 1.9.x
```

Jika `sudo.ws` belum terpasang:

```bash
sudo apt install sudo-ldap
sudo update-alternatives --set sudo /usr/bin/sudo.ws
```

### 4.4 Restart SSSD dan bersihkan cache

```bash
sudo systemctl restart sssd

# Bersihkan cache agar rule terbaru dari server langsung terambil
sudo sss_cache -E
sudo systemctl restart sssd
```

---

## 🔑 STEP 5: Automount Home Directory

Jika `--mkhomedir` digunakan, PAM otomatis membuat home directory saat login pertama.
Cek PAM config:

```bash
grep pam_mkhomedir /etc/pam.d/common-session
```

Jika belum ada, tambahkan:

```text
session required pam_mkhomedir.so skel=/etc/skel/ umask=0077
```

---

## 📁 STEP 6: Verifikasi

### 6.1 Test LDAP/Kerberos

```bash
getent passwd novel@rsud.internal
klist
```

### 6.2 Login user FreeIPA

```bash
su - novel
```

* Home directory `/home/novel` otomatis dibuat
* Permission 700, owner sesuai UID FreeIPA

### 6.3 Cek automount & SSSD

```bash
ls -ld /home/novel
```

### 6.4 Verifikasi sudo dari FreeIPA

```bash
# Sebagai user FreeIPA (bukan root)
su - novel

# Cek apakah rule dari FreeIPA terpakai
sudo -l
# Harus menampilkan daftar perintah yang diizinkan

# Cek laporan akses SSSD
sudo sssctl access-report rsud.internal
```

---

## 🧪 STEP 7: Troubleshooting

### 7.1 Error `Kerberos authentication failed: kinit: Cannot read password while getting initial credentials`

Penyebab paling umum:

1. **Port firewall client belum dibuka** → cek STEP 3.1
2. **Waktu tidak sinkron** → cek `chronyc sources`
3. **DNS tidak resolve** → cek `getent hosts rocky9-ipa.rsud.internal`
4. **Password admin salah/kedaluwarsa**
5. **Sisa state dari instalasi gagal sebelumnya**

Perbaikan:

```bash
# Bersihkan state gagal
sudo ipa-client-install --uninstall -U
sudo rm -f /var/lib/ipa-client/sysrestore/sysrestore.state

# Debug detail dengan trace Kerberos
KRB5_TRACE=/dev/stdout kinit admin
```

### 7.2 Error `sudo: Sorry, user <user> may not run sudo on <host>`

Ini gejala klasik `sudo-rs` di Ubuntu 24.04+/26.04, atau HBAC/sudo rule yang belum cocok.

**Diagnosis dari sisi server FreeIPA:**

```bash
# Uji HBAC untuk user & host spesifik
ipa hbactest --user=novel --host=sehat.rsud.internal --service=sudo
# Harus: Access granted: True

# Cek sudo rule
ipa sudorule-find --all
ipa sudorule-show <nama-rule> --all
```

**Diagnosis dari sisi client:**

```bash
# Cek implementasi sudo (yang paling sering jadi biang masalah)
sudo -V | head -1
# Jika 'sudo-rs' → ganti: sudo update-alternatives --set sudo /usr/bin/sudo.ws

# Cek SSSD membaca sudo
grep -E 'services|sudo_provider' /etc/sssd/sssd.conf
grep sudoers /etc/nsswitch.conf

# Cek rule yang di-cache SSSD
sudo sssctl access-report rsud.internal

# Bersihkan cache dan restart
sudo sss_cache -E
sudo systemctl restart sssd
```

### 7.3 Permission denied home directory

```bash
sudo chown <uid>:<gid> /home/novel
sudo chmod 700 /home/novel
```

### 7.4 SSSD error

```bash
journalctl -u sssd -n 50 --no-pager
sudo systemctl restart sssd
```

### 7.5 Kerberos gagal

```bash
kinit admin@RSUD.INTERNAL
klist
```
