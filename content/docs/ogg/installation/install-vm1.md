---
title: Instalasi OGG di VM1 (Source)
linkTitle: Instalasi OGG VM1
weight: 31
---

## Persiapan

Login ke VM1 sebagai user `oracle` dan pastikan installer sudah tersedia:

```bash
ls /home/oracle/fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/
```

---

## Langkah 1 — Buat Response File Instalasi

```bash
# Buat direktori untuk response file
mkdir -p /u01/app/ogg/response

# Buat response file instalasi OGG
cat > /u01/app/ogg/response/oggca.rsp << 'EOF'
oracle.install.responseFileVersion=/oracle/install/rspfmt_oggca_response_schema_v21_1_0
CONFIGURATION_OPTION=ADD
DEPLOYMENT_NAME=oggsource
ADMINISTRATOR_USER=ggadmin
ADMINISTRATOR_PASSWORD=@Rafi1234
SECURITY_ENABLED=false
SERVICEMANAGER_DEPLOYMENT_HOME=/u01/app/ogg_deployments
HOST_SERVICEMANAGER=oggrafi-src
PORT_SERVICEMANAGER=55000
CREATE_NEW_SERVICEMANAGER=true
OGG_SOFTWARE_HOME=/u01/app/ogg
OGG_DEPLOYMENT_HOME=/u01/app/ogg_depot/oggsource
ENV_TNS_ADMIN=/u01/app/oracle/product/19c/dbhome_1/network/admin
ADMINISTRATION_SERVER_ENABLED=true
PORT_ADMINSRVR=9001
DISTRIBUTION_SERVER_ENABLED=true
PORT_DISTSRVR=9002
RECEIVER_SERVER_ENABLED=true
PORT_RCVRSRVR=9003
METRICS_SERVER_ENABLED=false
PORT_PMSRVR=9004
UDP_PORT_PMSRVR=9005
PMSRVR_DATASTORE_TYPE=BDB
PMSRVR_DATASTORE_HOME=/u01/app/ogg_depot/oggsource/pmsrvr
OGG_SCHEMA=OGGADMIN
EOF
```

> **Perhatian Password:** Password harus memenuhi kriteria:
> - Minimal 8 karakter, maksimal 30 karakter
> - Minimal 1 huruf kecil [a..z]
> - Minimal 1 huruf besar [A..Z]
> - Minimal 1 angka [0..9]
> - Minimal 1 karakter spesial [- ! @ % & * . # _]

---

## Langkah 2 — Install OGG Software (Silent Mode)

```bash
cd /home/oracle/fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/

./runInstaller -silent \
  -responseFile /home/oracle/fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/response/oggcore.rsp \
  INVENTORY_LOCATION=/u01/app/oraInventory \
  SOFTWARE_LOCATION=/u01/app/ogg \
  START_MANAGER=false \
  INSTALL_OPTION=ORA21c
```

Output yang diharapkan:
```
Successfully Setup Software.
The installation of Oracle GoldenGate Services was successful.
```

---

## Langkah 3 — Jalankan Script Root

Login sebagai `root` dan jalankan:

```bash
su -
/u01/app/oraInventory/orainstRoot.sh
```

Output:
```
Changing permissions of /u01/app/oraInventory.
Adding read,write permissions for group.
The execution of the script is complete.
```

---

## Langkah 4 — Buat Direktori Deployment

Login kembali sebagai `oracle`:

```bash
su - oracle
mkdir -p /u01/app/ogg_deployments
mkdir -p /u01/app/ogg_depot/oggsource
```

---

## Langkah 5 — Setup Deployment (oggca.sh)

```bash
cd /u01/app/ogg/bin
./oggca.sh -silent -responseFile /u01/app/ogg/response/oggca.rsp
```

Output yang diharapkan:
```
Successfully Setup Software.
```

---

## Langkah 6 — Buka Port Firewall

Login sebagai `root`:

```bash
su -
firewall-cmd --permanent --add-port=55000/tcp
firewall-cmd --permanent --add-port=9001/tcp
firewall-cmd --permanent --add-port=9002/tcp
firewall-cmd --permanent --add-port=9003/tcp
firewall-cmd --permanent --add-port=9004/tcp
firewall-cmd --reload
firewall-cmd --list-ports
```

---

## Langkah 7 — Verifikasi Instalasi

```bash
# Cek Service Manager berjalan
ps -ef | grep ServiceManager | grep -v grep
```

Output yang diharapkan:
```
oracle   17282     1  0 17:54 ?  00:00:00 /u01/app/ogg/bin/ServiceManager --config ...
```

---

## Langkah 8 — Akses Web GUI

Buka browser dan akses:

```
http://192.168.245.132:55000
```

Login dengan:
- **Username:** `ggadmin`
- **Password:** `@Rafi1234`

Tampilan Service Manager menunjukkan deployment **oggsource** dengan 3 service Running:
- Administration Service (9001) — Running ✅
- Distribution Service (9002) — Running ✅
- Receiver Service (9003) — Running ✅

> **Instalasi Berhasil:** Jika Service Manager dapat diakses dan deployment oggsource berstatus Running, instalasi OGG di VM1 telah berhasil.

---

## Troubleshooting Instalasi

### Error: INS-75022 Install option was not specified

Tambahkan parameter `INSTALL_OPTION=ORA21c` pada perintah `runInstaller`.

### Error: INS-10105 Response file not valid

Pastikan format response file benar. Cek log di:
```bash
ls /u01/app/oraInventory/logs/OGGCAConfigActions*/
cat /u01/app/oraInventory/logs/OGGCAConfigActions*/*.log | grep -i "severe\|fatal"
```

### Error: root.sh not found

Pada OGG 21c Microservices, tidak ada `root.sh`. Cukup jalankan `orainstRoot.sh` saja.
