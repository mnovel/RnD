# 🐳 Panduan Instalasi Docker di Ubuntu 24.04

Dokumentasi lengkap untuk menginstal Docker Engine dan Docker Compose Plugin di Ubuntu 24.04 (Noble Numbat).

---

## 🧰 Prasyarat
- Ubuntu 24.04 LTS
- Akses user dengan hak `sudo`
- Koneksi internet aktif
- Disarankan minimal 2GB RAM

---

## 🔄 STEP 1: Update Sistem
Pastikan sistem dalam kondisi terbaru.

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 📦 STEP 2: Instalasi Dependensi
Install paket pendukung yang dibutuhkan Docker.

```bash
sudo apt install -y ca-certificates curl gnupg
```

---

## 🐳 STEP 3: Instalasi Docker Engine
Gunakan script resmi Docker untuk instalasi otomatis.

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

---

## ▶️ STEP 4: Enable dan Jalankan Docker
Aktifkan Docker agar otomatis berjalan saat boot.

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## ✅ STEP 5: Verifikasi Instalasi Docker
Cek status service Docker.

```bash
sudo systemctl status docker
```

Output yang diharapkan:
```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled)
     Active: active (running)
```

---

## 🧩 STEP 6: Instal Docker Compose Plugin
Docker Compose versi terbaru tersedia sebagai plugin resmi.

```bash
sudo apt install -y docker-compose-plugin
```

Cek versi Docker Compose:
```bash
docker compose version
```

---

## 👤 STEP 7: Menjalankan Docker Tanpa Sudo (Opsional)
Tambahkan user ke grup docker.

```bash
sudo usermod -aG docker $USER
```

Logout lalu login kembali agar perubahan berlaku.

---

## 🧪 STEP 8: Uji Coba Docker
Jalankan container test untuk memastikan Docker berjalan normal.

```bash
docker run hello-world
```

Jika muncul pesan **Hello from Docker!**, maka instalasi berhasil 🎉

---

## 🛠️ STEP 9: Perintah Docker Penting
| Perintah | Deskripsi |
|--------|----------|
| `docker ps` | Melihat container yang sedang berjalan |
| `docker ps -a` | Melihat semua container |
| `docker images` | Melihat daftar image |
| `docker pull nginx` | Download image |
| `docker run nginx` | Menjalankan container |
| `docker stop <id>` | Menghentikan container |
| `docker rm <id>` | Menghapus container |
| `docker compose up -d` | Menjalankan docker compose |

---

## 🔐 STEP 10: Keamanan Dasar Docker
1. Jangan expose Docker API (`2375`) ke publik
2. Gunakan user non-root
3. Batasi port container yang diexpose
4. Update Docker secara berkala
5. Gunakan firewall (UFW / iptables)

---

## 🗑️ STEP 11: Uninstall Docker
Jika ingin menghapus Docker sepenuhnya:

```bash
sudo apt remove --purge docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
sudo apt autoremove -y
sudo rm -rf /var/lib/docker /var/lib/containerd
```

---

## 🆘 Troubleshooting
- **Docker tidak bisa jalan**  
  ```bash
  sudo journalctl -u docker.service
  ```

- **Permission denied**  
  Pastikan user sudah masuk grup docker dan logout/login ulang

- **Compose tidak ditemukan**  
  Gunakan:
  ```bash
  docker compose version
  ```

---

## 📚 Referensi
- https://docs.docker.com/engine/install/
- https://docs.docker.com/compose/
- https://get.docker.com/
