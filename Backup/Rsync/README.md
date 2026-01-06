# 🚀 Rsync Installation Guide for Ubuntu 24.04

Complete documentation for installing and configuring rsync on Ubuntu 24.04 (Noble Numbat).

## 🧰 Prerequisites
- Ubuntu 24.04 LTS
- User with sudo privileges
- Active internet connection

## 🔄 STEP 1: System Update
```bash
sudo apt update && sudo apt upgrade -y
```

## 📦 STEP 2: Rsync Installation
```bash
sudo apt install rsync -y
```

## ✅ STEP 3: Verify Installation
```bash
rsync --version
```
Expected output:
```
rsync  version 3.2.7  protocol version 31
...
```

## ⚙️ STEP 4: Basic Configuration (Optional)
### 4.1 Create rsync daemon configuration file:
```bash
sudo nano /etc/rsyncd.conf
```

### 4.2 Basic configuration example:
```ini
[backup]
    path = /path/to/backup
    comment = Backup Directory
    read only = no
    list = yes
    uid = root
    gid = root
```

## 🚦 STEP 5: Run as Service (Optional)
```bash
sudo systemctl enable rsync
sudo systemctl start rsync
sudo systemctl status rsync
```

## 🛠️ STEP 6: Common Rsync Commands
| Command | Description |
|---------|-------------|
| `rsync -avz /source/ /destination/` | Local synchronization |
| `rsync -avz -e ssh user@remote:/remote/dir/ /local/dir/` | Download from remote |
| `rsync -avz -e ssh /local/dir/ user@remote:/remote/dir/` | Upload to remote |
| `rsync -avz --delete /source/ /destination/` | Sync with deletion of extraneous files |
| `rsync -avz --dry-run --delete -e ssh /source/ /destination/` | Dry-run remote sync with deletion preview |

## 🔍 STEP 7: Important Rsync Options
| Option | Function |
|--------|----------|
| `-a` | Archive mode (preserve permissions) |
| `-v` | Verbose output |
| `-z` | Compression during transfer |
| `-h` | Human-readable output |
| `--progress` | Show transfer progress |
| `--exclude` | Exclude specific files/directories |
| `--delete` | Delete files in destination not present in source |
| `--dry-run` | Simulation mode (no real changes) |

## 🗑️ STEP 8: Uninstall Rsync
```bash
sudo apt remove rsync -y
```

## 📌 STEP 9: Troubleshooting
- **Permission denied**: Ensure user has access to directories
- **Connection refused**: Check if rsync daemon is running (`sudo systemctl status rsync`)
- **SSH issues**: For remote transfers, verify SSH is working properly

## 🔗 References
- Manual page: `man rsync`
- Ubuntu documentation: https://help.ubuntu.com/community/rsync
