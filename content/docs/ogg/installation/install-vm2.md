---
title: Instalasi OGG di VM2 (Target)
linkTitle: Instalasi OGG VM2
weight: 32
---

> **Catatan:** Proses instalasi OGG di VM2 hampir sama dengan VM1. Perbedaan utama ada pada nama deployment dan direktori.

## Langkah 1 — Buat Response File

Login ke VM2 sebagai `oracle`:

```bash
mkdir -p /u01/app/ogg/response

cat > /u01/app/ogg/response/oggca.rsp << 'EOF'
oracle.install.responseFileVersion=/oracle/install/rspfmt_oggca_response_schema_v21_1_0
CONFIGURATION_OPTION=ADD
DEPLOYMENT_NAME=oggtarget
ADMINISTRATOR_USER=ggadmin
ADMINISTRATOR_PASSWORD=@Rafi1234
SECURITY_ENABLED=false
SERVICEMANAGER_DEPLOYMENT_HOME=/u01/app/ogg_deployments
HOST_SERVICEMANAGER=oggrafi-tgt
PORT_SERVICEMANAGER=55000
CREATE_NEW_SERVICEMANAGER=true
OGG_SOFTWARE_HOME=/u01/app/ogg
OGG_DEPLOYMENT_HOME=/u01/app/ogg_depot/oggtarget
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
PMSRVR_DATASTORE_HOME=/u01/app/ogg_depot/oggtarget/pmsrvr
OGG_SCHEMA=OGGADMIN
EOF
```

---

## Langkah 2 — Install OGG Software

```bash
cd /home/oracle/fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/

./runInstaller -silent \
  -responseFile /home/oracle/fbo_ggs_Linux_x64_Oracle_services_shiphome/Disk1/response/oggcore.rsp \
  INVENTORY_LOCATION=/u01/app/oraInventory \
  SOFTWARE_LOCATION=/u01/app/ogg \
  START_MANAGER=false \
  INSTALL_OPTION=ORA21c
```

---

## Langkah 3 — Jalankan Script Root

```bash
su -
/u01/app/oraInventory/orainstRoot.sh
```

---

## Langkah 4 — Buat Direktori Deployment

```bash
su - oracle
mkdir -p /u01/app/ogg_deployments
mkdir -p /u01/app/ogg_depot/oggtarget
```

---

## Langkah 5 — Setup Deployment

```bash
cd /u01/app/ogg/bin
./oggca.sh -silent -responseFile /u01/app/ogg/response/oggca.rsp
```

---

## Langkah 6 — Buka Port Firewall

```bash
su -
firewall-cmd --permanent --add-port=55000/tcp
firewall-cmd --permanent --add-port=9001/tcp
firewall-cmd --permanent --add-port=9002/tcp
firewall-cmd --permanent --add-port=9003/tcp
firewall-cmd --permanent --add-port=9004/tcp
firewall-cmd --reload
```

---

## Langkah 7 — Verifikasi

Akses Web GUI VM2 dari browser:

```
http://192.168.245.133:55000
```

> **Instalasi Berhasil:** Jika deployment **oggtarget** berstatus Running, instalasi OGG di VM2 telah berhasil.
