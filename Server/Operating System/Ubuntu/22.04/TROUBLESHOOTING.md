# TROUBLESHOOTING.md — Ubuntu Server 22.04 LTS

Dokumen ini berisi panduan troubleshooting umum server Ubuntu 22.04.

---

## 1. Cek Status Service

```bash
systemctl status ssh
systemctl status rsyslog
systemctl status ufw
```

---

## 2. Masalah Network

```bash
ip a
ip route
ping 8.8.8.8
```

---

## 3. SSH Tidak Bisa Login

Cek:

- User ada
- Permission `.ssh`

```bash
ls -la ~/.ssh/
```

Log SSH:

```bash
journalctl -u ssh -xe
```

---

## 4. Disk Penuh

```bash
df -h
du -sh /var/log/*
```

---

## 5. Firewall Memblokir Akses

```bash
sudo ufw status verbose
sudo iptables -L -n
```

---

## 6. Log Penting

- `/var/log/syslog`
- `/var/log/auth.log`
- `/var/log/kern.log`

---

## 7. Debug Cepat

```bash
journalctl -p err -xb
```
