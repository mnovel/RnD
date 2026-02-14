# HARDENING.md — Ubuntu Server 24.04 LTS

Dokumen ini berisi **hardening dasar–menengah** untuk Ubuntu Server 24.04 (production-ready).

---

## 1. SSH Hardening

Edit konfigurasi SSH:

```bash
sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf
```

Pastikan:

```
PermitRootLogin yes
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
AllowTcpForwarding no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## 2. Firewall (UFW)

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
```

Cek:

```bash
sudo ufw status verbose
```

---

## 3. Fail2Ban

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

## 4. Kernel & Network Hardening (sysctl)

```bash
sudo nano /etc/sysctl.d/99-hardening.conf
```

Isi:

```
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
kernel.randomize_va_space = 2
fs.protected_symlinks = 1
fs.protected_hardlinks = 1
```

Apply:

```bash
sudo sysctl --system
```

---

## 5. Automatic Security Update

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

---

## 6. Logging & Audit

```bash
sudo apt install auditd audispd-plugins -y
sudo systemctl enable auditd
```

---

# 7. User & Password Policy Hardening

## 7.1 Set Default Password Policy (Global)

Edit file:

```bash
sudo nano /etc/login.defs
```

Pastikan:

```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   7
```

Artinya:

* Password expired 90 hari
* Minimal 7 hari sebelum bisa diganti lagi
* Warning 7 hari sebelum expired

---

## 7.2 Buat User Admin Non-Root

```bash
sudo adduser novel
sudo usermod -aG sudo novel
```

---

## 7.3 Paksa Ganti Password Saat Login Pertama

```bash
sudo chage -d 0 novel
```

---

## 7.4 Setup SSH Key Login

```bash
sudo mkdir -p /home/novel/.ssh
sudo touch /home/novel/.ssh/authorized_keys
sudo chmod 700 /home/novel/.ssh
sudo chmod 600 /home/novel/.ssh/authorized_keys
sudo chown -R novel:novel /home/novel/.ssh
```

Lalu masukkan public key ke file:

```bash
sudo nano /home/novel/.ssh/authorized_keys
```

---

## 8. Additional Hardening (Recommended)

* Disable unused services (`systemctl list-unit-files`)
* Gunakan SSH key dengan passphrase
* Batasi user dalam group sudo
* Pertimbangkan disable root SSH login setelah admin user siap:

  ```
  PermitRootLogin no
  ```

---

## 9. Checklist Hardening

* [x] SSH key only
* [x] Firewall aktif
* [x] Fail2Ban
* [x] Kernel hardening
* [x] Auto security update
* [x] Audit logging
* [x] Password policy 90 hari
* [x] Non-root sudo user
* [x] Force change password first login
