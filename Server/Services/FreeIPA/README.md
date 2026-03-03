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

### 3.1 Update sistem

```bash
sudo apt update && sudo apt upgrade -y
```

### 3.2 Install FreeIPA client

```bash
sudo apt install freeipa-client -y
```

### 3.3 Join Client ke FreeIPA Server

```bash
sudo ipa-client-install --mkhomedir
```

> Pilihan `--mkhomedir` membuat **home directory otomatis** saat user login.
> Ikuti prompt:
>
> * Nama domain: `rsud.internal`
> * Server IPA: `rocky9-ipa.rsud.internal`
> * Konfirmasi Kerberos realm

---

## 🖥️ STEP 4: Konfigurasi SSSD (Client)

Cek `/etc/sssd/sssd.conf`:

```ini
[domain/rsud.internal]
id_provider = ipa
auth_provider = ipa
access_provider = ipa
cache_credentials = True
override_homedir = /home/%u
default_shell = /bin/bash

[sssd]
services = nss, pam
config_file_version = 2
domains = rsud.internal
```

Restart SSSD:

```bash
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

---

## 🧪 STEP 7: Troubleshooting

* **Permission denied home directory**

  ```bash
  sudo chown <uid>:<gid> /home/novel
  sudo chmod 700 /home/novel
  ```
* **SSSD error**

  ```bash
  journalctl -u sssd -n 50 --no-pager
  sudo systemctl restart sssd
  ```
* **Kerberos gagal**

  ```bash
  kinit admin@RSUD.INTERNAL
  klist
  ```
