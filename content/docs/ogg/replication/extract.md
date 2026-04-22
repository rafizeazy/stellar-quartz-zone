---
title: Extract (Source — VM1)
linkTitle: Extract (VM1)
weight: 51
---

Extract adalah komponen OGG yang bertugas **membaca perubahan data** dari redo log Oracle Database source dan menulis hasilnya ke trail file.

---

## Buat Extract di VM1

### Akses Administration Service

```
http://192.168.245.132:9001
```

Login: `ggadmin` / `@Rafi1234`

### Langkah Membuat Extract

1. Di halaman **Overview**, klik **+** di bagian **Extracts**
2. Pilih type **Integrated Extract** → klik **Next**
3. Isi form **Basic Information**:

| Field | Nilai |
|---|---|
| Process Name | `EXTEMP` |
| Description | `Extract data dari VM1 (SOURCE)` |
| Intent | `Unidirectional` |
| Credential Domain | `OracleGoldenGate` |
| Credential Alias | `srcdb` |
| Trail Name | `et` |
| Begin | `Now` |

4. Klik **Next**

5. Isi **Parameter File**:

```
USERIDALIAS srcdb DOMAIN OracleGoldenGate
EXTTRAIL et
TABLE RAFI.EMPLOYEES;
TABLE RAFI.TRANSAKSI_QRIS;
REPORTCOUNT EVERY 30 MINUTES, RATE
DISCARDFILE ./dirrpt/EXTEMP.dsc, APPEND, MEGABYTES 100
```

> **Penting — EXTTRAIL:** Gunakan `EXTTRAIL et` (tanpa path lengkap dan tanpa `./dirdat/`). Ini adalah hal kritis yang memastikan Distribution Path dapat membaca trail file dengan benar.

6. Klik **Create and Run**

---

## Verifikasi Extract

### Cek Status

Pastikan EXTEMP berstatus **Running** di halaman Overview.

### Cek Statistics

1. Klik **EXTEMP** → tab **Statistics**
2. Setelah ada transaksi di database source, tabel `RAFI.EMPLOYEES` dan `RAFI.TRANSAKSI_QRIS` akan muncul dengan jumlah Inserts/Updates/Deletes

### Cek via CLI

```bash
# Cek trail file sudah ada dan ada datanya
ls -la /u01/app/ogg_depot/oggsource/var/lib/data/dirdat/et*
```

---

## Perintah Penting via adminclient

```bash
/u01/app/ogg/bin/adminclient
```

```
connect http://192.168.245.132:55000 deployment oggsource as ggadmin password @Rafi1234

# Cek info exttrail
info exttrail *

# Cek info semua process
info all

exit
```

---

## Troubleshooting Extract

### Error: PROCESS ABENDING — Unable to connect to database

Pastikan:

1. Credential alias `srcdb` sudah terdaftar dan bisa konek
2. User `GGADMIN` memiliki privilege OGG:
    ```sql
    EXEC DBMS_GOLDENGATE_AUTH.GRANT_ADMIN_PRIVILEGE('GGADMIN');
    ```
3. `enable_goldengate_replication=TRUE`

### Error: Trail is not assigned to extract

Jalankan via adminclient:
```
ADD EXTTRAIL et, EXTRACT EXTEMP
```

### Warning: streams_pool_size not configured

```sql
ALTER SYSTEM SET streams_pool_size=200M SCOPE=BOTH;
```
