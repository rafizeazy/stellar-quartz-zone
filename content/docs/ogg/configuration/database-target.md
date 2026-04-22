---
title: Konfigurasi Database Target (VM2)
linkTitle: Database Target (VM2)
weight: 42
---

## Langkah 1 — Aktifkan Supplemental Logging dan GoldenGate Replication

Login sebagai `oracle` di VM2:

```bash
sqlplus / as sysdba << EOF
-- Aktifkan supplemental logging
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;

-- Aktifkan GoldenGate replication
ALTER SYSTEM SET enable_goldengate_replication=TRUE SCOPE=BOTH;

-- Verifikasi
SELECT LOG_MODE, SUPPLEMENTAL_LOG_DATA_MIN FROM V\$DATABASE;
SHOW PARAMETER enable_goldengate_replication;
EOF
```

---

## Langkah 2 — Grant Privilege ke User GGADMIN

```bash
sqlplus / as sysdba << EOF
EXEC DBMS_GOLDENGATE_AUTH.GRANT_ADMIN_PRIVILEGE('GGADMIN');

SELECT USERNAME, ACCOUNT_STATUS FROM DBA_USERS WHERE USERNAME='GGADMIN';
EOF
```

---

## Langkah 3 — Verifikasi Listener

```bash
lsnrctl status
```

Jika tidak ada service terdaftar:
```bash
sqlplus / as sysdba << EOF
ALTER SYSTEM REGISTER;
EOF

sleep 5
lsnrctl status
```

---

## Langkah 4 — Buat Tabel yang Sama di Target

> **Penting:** Struktur tabel di VM2 (Target) **harus sama persis** dengan VM1 (Source).

```bash
sqlplus rafi/Rafi1234@ORCL << EOF
-- Buat tabel EMPLOYEES
CREATE TABLE RAFI.EMPLOYEES (
  ID     NUMBER NOT NULL,
  NAME   VARCHAR2(100),
  SALARY NUMBER,
  PRIMARY KEY (ID)
);

-- Buat tabel TRANSAKSI_QRIS
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

-- Verifikasi
SELECT TABLE_NAME FROM USER_TABLES;
EXIT
EOF
```

---

## Langkah 5 — Buat Checkpoint Table

Checkpoint table digunakan OGG untuk menyimpan posisi replikasi Replicat.

```bash
# Buat checkpoint table via Web GUI VM2
# http://192.168.245.133:9001
# Configuration → Credentials → Connect (tgtdb) → Checkpoint → +
```

Atau drop manual jika perlu buat ulang:
```bash
sqlplus ggadmin/Rafi1234@ORCL << EOF
DROP TABLE GGADMIN.CHECKPOINTTABLE;
EXIT
EOF
```

Lalu buat kembali via Web GUI agar struktur tabel sesuai format OGG.

> **Tips:** Checkpoint table **harus dibuat via Web GUI OGG**, bukan manual via SQL, agar strukturnya sesuai dengan yang diharapkan OGG.
