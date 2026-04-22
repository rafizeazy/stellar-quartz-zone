---
title: Konfigurasi Database Source (VM1)
linkTitle: Database Source (VM1)
weight: 41
---

## Langkah 1 — Aktifkan Supplemental Logging dan GoldenGate Replication

Login sebagai `oracle` di VM1:

```bash
sqlplus / as sysdba << EOF
-- Aktifkan supplemental logging (wajib untuk OGG)
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;

-- Aktifkan GoldenGate replication
ALTER SYSTEM SET enable_goldengate_replication=TRUE SCOPE=BOTH;

-- Set streams pool size (opsional tapi direkomendasikan)
ALTER SYSTEM SET streams_pool_size=200M SCOPE=BOTH;

-- Verifikasi
SELECT LOG_MODE, SUPPLEMENTAL_LOG_DATA_MIN FROM V\$DATABASE;
SHOW PARAMETER enable_goldengate_replication;
EOF
```

Output yang diharapkan:
```
LOG_MODE     SUPPLEME
------------ --------
ARCHIVELOG   YES

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
enable_goldengate_replication        boolean     TRUE
```

> **Penting:** Database **harus** dalam mode ARCHIVELOG. Jika belum, aktifkan terlebih dahulu:
> ```sql
> SHUTDOWN IMMEDIATE;
> STARTUP MOUNT;
> ALTER DATABASE ARCHIVELOG;
> ALTER DATABASE OPEN;
> ```

---

## Langkah 2 — Grant Privilege ke User GGADMIN

```bash
sqlplus / as sysdba << EOF
-- Grant privilege OGG ke user GGADMIN
EXEC DBMS_GOLDENGATE_AUTH.GRANT_ADMIN_PRIVILEGE('GGADMIN');

-- Verifikasi status user
SELECT USERNAME, ACCOUNT_STATUS FROM DBA_USERS WHERE USERNAME='GGADMIN';
EOF
```

Output yang diharapkan:
```
USERNAME             ACCOUNT_STATUS
-------------------- --------------------------------
GGADMIN              OPEN
```

---

## Langkah 3 — Verifikasi Listener

```bash
# Cek listener berjalan dan database terdaftar
lsnrctl status
```

Output yang diharapkan:
```
Service "ORCL" has 1 instance(s).
  Instance "ORCL", status READY, has 1 handler(s) for this service...
```

Jika listener berjalan tapi tidak ada service:
```bash
sqlplus / as sysdba << EOF
ALTER SYSTEM REGISTER;
EOF

sleep 5
lsnrctl status
```

---

## Langkah 4 — Test Koneksi Database

```bash
# Test koneksi dengan user GGADMIN
sqlplus ggadmin/Rafi1234@ORCL
```

Jika berhasil masuk SQL*Plus, koneksi database berhasil.

---

## Langkah 5 — Buat Tabel untuk Replikasi

```bash
sqlplus rafi/Rafi1234@ORCL << EOF
-- Contoh: Tabel EMPLOYEES
CREATE TABLE RAFI.EMPLOYEES (
  ID     NUMBER NOT NULL,
  NAME   VARCHAR2(100),
  SALARY NUMBER,
  PRIMARY KEY (ID)
);

-- Contoh: Tabel TRANSAKSI_QRIS
CREATE TABLE RAFI.TRANSAKSI_QRIS (
  ID_TRANSAKSI    VARCHAR2(50)   NOT NULL,
  ID_MERCHANT     VARCHAR2(50)   NOT NULL,
  NAMA_MERCHANT   VARCHAR2(100),
  ID_PELANGGAN    VARCHAR2(50),
  NAMA_PELANGGAN  VARCHAR2(100),
  JUMLAH          NUMBER(15,2)   NOT NULL,
  STATUS          VARCHAR2(20)   DEFAULT 'PENDING',
  WAKTU_TRANSAKSI TIMESTAMP      DEFAULT SYSTIMESTAMP,
  KETERANGAN      VARCHAR2(255),
  PRIMARY KEY (ID_TRANSAKSI)
);

-- Verifikasi tabel
SELECT TABLE_NAME FROM USER_TABLES WHERE OWNER='RAFI';
EXIT
EOF
```
