# HARDENING.md — Ubuntu Server 24.04 LTS

Dokumen ini berisi **hardening dasar–menengah** untuk Ubuntu Server 24.04 (production-ready).

---

## 1. SSH Hardening

Edit konfigurasi SSH:

```bash
sudo nano /etc/ssh/sshd_config
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

## 7. Additional Hardening (Recommended)

- Disable unused services
- Gunakan SSH key dengan passphrase
- Batasi user sudo

---

## 8. Checklist Hardening

- [x] SSH key only
- [x] Firewall aktif
- [x] Fail2Ban
- [x] Kernel hardening
- [x] Auto security update
- [x] Audit logging
