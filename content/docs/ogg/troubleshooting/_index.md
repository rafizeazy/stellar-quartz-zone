---
title: Troubleshooting
linkTitle: Troubleshooting
weight: 60
sidebar:
  open: true
---

Kumpulan error yang ditemui selama instalasi dan konfigurasi OGG 21c beserta solusinya.

---

## Instalasi

### INS-75022: Install option was not specified

**Penyebab:** Parameter `INSTALL_OPTION` tidak ditentukan.

**Solusi:** Tambahkan `INSTALL_OPTION=ORA21c` pada perintah `runInstaller`:
```bash
./runInstaller -silent \
  -responseFile /path/to/oggcore.rsp \
  INSTALL_OPTION=ORA21c \
  ...
```

---

### INS-10105: Response file not valid

**Penyebab:** Format atau isi response file salah.

**Solusi:**
```bash
# Cek log detail
cat /u01/app/oraInventory/logs/OGGCAConfigActions*/*.log | grep -i "severe\|fatal\|missing"
```

Pastikan format response file adalah **plain text properties**, bukan XML.

---

### INS-85105: Service Manager deployment home invalid

**Penyebab:** `SERVICEMANAGER_DEPLOYMENT_HOME` berada di dalam `OGG_SOFTWARE_HOME`.

**Solusi:** Gunakan direktori di luar `/u01/app/ogg`:
```
SERVICEMANAGER_DEPLOYMENT_HOME=/u01/app/ogg_deployments
OGG_DEPLOYMENT_HOME=/u01/app/ogg_depot/oggsource
```

---

### INS-85080: Invalid password

**Penyebab:** Password tidak memenuhi strong password policy.

**Solusi:** Gunakan password yang mengandung huruf besar, huruf kecil, angka, dan karakter spesial. Contoh: `@Rafi1234`

---

## Koneksi Database

### ORA-12545: TNS destination host unreachable

**Penyebab:** IP di tnsnames.ora tidak sesuai dengan IP aktual VM.

**Solusi:**
```bash
# Cek IP aktual
ip addr show ens33 | grep "inet "

# Update tnsnames.ora
cat > /u01/app/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora << 'EOF'
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = <IP_AKTUAL>(PORT = 1521))
    (CONNECT_DATA =
      (SERVICE_NAME = ORCL)
    )
  )
EOF
```

---

### TNS-03505: Failed to resolve name

**Penyebab:** Ada spasi di awal baris `ORCL =` pada tnsnames.ora.

**Solusi:** Tulis ulang tnsnames.ora pastikan `ORCL =` dimulai dari kolom pertama (tanpa spasi).

---

### OGG-08110: Login failed — ORA-00942: table or view does not exist

**Penyebab:** User OGG tidak punya privilege yang cukup.

**Solusi:**
```sql
EXEC DBMS_GOLDENGATE_AUTH.GRANT_ADMIN_PRIVILEGE('GGADMIN');
```

---

### The listener supports no services

**Penyebab:** Database tidak terdaftar ke listener.

**Solusi:**
```bash
sqlplus / as sysdba << EOF
ALTER SYSTEM REGISTER;
EOF
sleep 5
lsnrctl status
```

---

## Extract

### PROCESS ABENDING — Unable to connect to database

**Penyebab:** Credential alias tidak bisa konek ke database.

**Langkah debug:**

1. Test tnsping:
    ```bash
    tnsping ORCL
    ```
2. Test login sqlplus:
    ```bash
    sqlplus ggadmin/Rafi1234@ORCL
    ```
3. Test credential di Web GUI → Configuration → Credentials → klik 🔌

---

### Trail is not assigned to extract

**Penyebab:** Trail file tidak terdaftar ke Extract.

**Solusi:** Via adminclient:
```bash
/u01/app/ogg/bin/adminclient
connect http://192.168.245.132:55000 deployment oggsource as ggadmin password @Rafi1234
ADD EXTTRAIL et, EXTRACT EXTEMP
exit
```

---

## Distribution Path

### LCR Read from Trails tetap 0

**Penyebab:** Source URI tidak mengandung `extract=EXTEMP`.

**Solusi:** Delete dan buat ulang Distribution Path. Pastikan field **Source** dipilih dari **dropdown EXTEMP**, bukan diketik manual.

Verifikasi Source URI:
```
trail://192.168.245.132:9002/services/v2/sources?trail=et&extract=EXTEMP
```

---

### DB Name dan Extract UNKNOWN

**Penyebab:** Distribution Path tidak terhubung ke Extract.

**Solusi:** Sama dengan LCR Read from Trails tetap 0 — buat ulang Distribution Path dengan memilih Source dari dropdown.

---

## Replicat

### Data tidak muncul di VM2 meskipun Replicat Running

**Langkah debug:**

1. Cek trail file `rt` di VM2 tidak kosong:
    ```bash
    ls -la /u01/app/ogg_depot/oggtarget/var/lib/data/rt*
    ```

2. Cek Checkpoint Replicat — apakah Sequence berubah?

3. Cek trail name Replicat — harus `rt`, bukan `et`

4. Cek report Replicat:
    ```bash
    cat $(find /u01/app/ogg_depot/oggtarget -name "REPEMP*.rpt") | tail -50
    ```

---

## Monitoring Umum

### Cek semua process OGG berjalan

```bash
# VM1
ps -ef | grep -E "ServiceManager|adminsrvr|distsrvr|recvsrvr|extract" | grep -v grep

# VM2
ps -ef | grep -E "ServiceManager|adminsrvr|distsrvr|recvsrvr|replicat" | grep -v grep
```

### Cek log Service Manager

```bash
find /u01/app/ogg_deployments -name "*.log" | xargs ls -lrt | tail -5
```

### Perintah adminclient yang berguna

```bash
/u01/app/ogg/bin/adminclient
connect http://192.168.245.132:55000 deployment oggsource as ggadmin password @Rafi1234

info all          # Status semua process
info exttrail *   # Info trail file
info extract *    # Info detail Extract
exit
```