---
title: System Requirements
linkTitle: System Requirements
weight: 22
sidebar:
  open: true
---

## Spesifikasi VM yang Digunakan

| Spesifikasi | VM1 (Source) | VM2 (Target) |
|---|---|---|
| **Hostname** | oggrafi-src | oggrafi-tgt |
| **IP Address** | 192.168.245.132 | 192.168.245.133 |
| **OS** | Oracle Linux 7 | Oracle Linux 7 |
| **RAM** | Minimal 4 GB | Minimal 4 GB |
| **Storage** | Minimal 50 GB | Minimal 50 GB |
| **CPU** | Minimal 2 Core | Minimal 2 Core |

---

## Prasyarat Software

### Oracle Database 19c
Pastikan Oracle Database 19c sudah terinstall dan berjalan di kedua VM sebelum instalasi OGG.

```bash
# Verifikasi Oracle Database berjalan
ps -ef | grep pmon | grep -v grep
```

Output yang diharapkan:
```
oracle   32258     1  0 17:05 ?        00:00:01 ora_pmon_ORCL
```

### Listener Oracle
```bash
# Verifikasi listener berjalan
lsnrctl status
```

Output yang diharapkan:
```
Service "ORCL" has 1 instance(s).
  Instance "ORCL", status READY, has 1 handler(s) for this service...
```

---

## Prasyarat Sistem Operasi

### Disk Space
```bash
# Cek temp space (minimal 120 MB)
df -h /tmp

# Cek swap space (minimal 150 MB)
free -m
```

### User Oracle
Pastikan user `oracle` sudah ada di sistem:
```bash
id oracle
```

### Direktori yang Diperlukan
```bash
# Buat direktori instalasi OGG
mkdir -p /u01/app/ogg
mkdir -p /u01/app/oraInventory
mkdir -p /u01/app/ogg_deployments
mkdir -p /u01/app/ogg_depot/oggsource   # Khusus VM1
mkdir -p /u01/app/ogg_depot/oggtarget   # Khusus VM2

# Set ownership
chown -R oracle:oinstall /u01/app/ogg
chown -R oracle:oinstall /u01/app/oraInventory
chown -R oracle:oinstall /u01/app/ogg_deployments
chown -R oracle:oinstall /u01/app/ogg_depot
```

---

## File Installer OGG

Download file installer OGG 21c dari Oracle:

| File | Keterangan | Link Download |
|---|---|---|
| `fbo_ggs_Linux_x64_Oracle_services_shiphome.zip` | OGG 21c Microservices untuk Oracle | [Oracle Software Delivery Cloud](https://edelivery.oracle.com/) |

> **Cara Download:**
> 1. Buka [edelivery.oracle.com](https://edelivery.oracle.com/)
> 2. Login dengan akun Oracle
> 3. Cari **"Oracle GoldenGate"**
> 4. Pilih versi **21.3** untuk platform **Linux x86-64**
> 5. Download file `.zip`

Setelah download, extract ke VM:
```bash
# Extract installer (lakukan sebagai user oracle)
cd /home/oracle
unzip fbo_ggs_Linux_x64_Oracle_services_shiphome.zip
ls fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/
```

---

## Port yang Diperlukan

Pastikan port berikut terbuka di firewall kedua VM:

| Port | Service | Keterangan |
|---|---|---|
| 55000 | Service Manager | Manajemen pusat OGG |
| 9001 | Administration Service | Kelola Extract/Replicat |
| 9002 | Distribution Service | Kirim trail file |
| 9003 | Receiver Service | Terima trail file |
| 9004 | Performance Metrics | Monitor performa |
| 1521 | Oracle Listener | Koneksi database |

```bash
# Buka port di firewall (jalankan sebagai root)
firewall-cmd --permanent --add-port=55000/tcp
firewall-cmd --permanent --add-port=9001/tcp
firewall-cmd --permanent --add-port=9002/tcp
firewall-cmd --permanent --add-port=9003/tcp
firewall-cmd --permanent --add-port=9004/tcp
firewall-cmd --reload
firewall-cmd --list-ports
```

---

## Variabel Environment Oracle

Pastikan variabel environment Oracle sudah dikonfigurasi untuk user `oracle`:

```bash
# Tambahkan ke /home/oracle/.bash_profile
export ORACLE_HOME=/u01/app/oracle/product/19c/dbhome_1
export ORACLE_SID=ORCL
export PATH=$ORACLE_HOME/bin:$PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin
```

> **Tips:** Verifikasi variabel environment dengan perintah:
> ```bash
> echo $ORACLE_HOME
> echo $ORACLE_SID
> echo $TNS_ADMIN
> ```
