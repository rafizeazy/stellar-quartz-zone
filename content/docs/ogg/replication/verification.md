---
title: Verifikasi Replikasi
linkTitle: Verifikasi Replikasi
weight: 54
---

Setelah semua komponen (Extract, Distribution, Replicat) berjalan, lakukan verifikasi bahwa data benar-benar terreplikasi dari VM1 ke VM2.

---

## Test Replikasi — Insert Data Baru

### Langkah 1 — Insert Data di VM1 (Source)

Login sebagai `oracle` di VM1:

```bash
sqlplus rafi/Rafi1234@ORCL << EOF
-- Insert ke tabel EMPLOYEES
INSERT INTO RAFI.EMPLOYEES (ID, NAME, SALARY) VALUES (1001, 'Test Replikasi', 9000000);
COMMIT;

-- Insert ke tabel TRANSAKSI_QRIS
INSERT INTO RAFI.TRANSAKSI_QRIS
  (ID_TRANSAKSI, ID_MERCHANT, NAMA_MERCHANT, ID_PELANGGAN, NAMA_PELANGGAN, JUMLAH, STATUS)
VALUES
  ('TRX001', 'MCH001', 'Warung Makan Pak Budi', 'PLG001', 'Rafi', 50000, 'SUCCESS');
COMMIT;

-- Verifikasi data di VM1
SELECT * FROM RAFI.EMPLOYEES WHERE ID = 1001;
SELECT * FROM RAFI.TRANSAKSI_QRIS WHERE ID_TRANSAKSI = 'TRX001';
EXIT
EOF
```

### Langkah 2 — Cek Data di VM2 (Target)

Login sebagai `oracle` di VM2 (tunggu beberapa detik):

```bash
sqlplus rafi/Rafi1234@ORCL << EOF
-- Cek data terreplikasi di VM2
SELECT * FROM RAFI.EMPLOYEES WHERE ID = 1001;
SELECT * FROM RAFI.TRANSAKSI_QRIS WHERE ID_TRANSAKSI = 'TRX001';
EXIT
EOF
```

Output yang diharapkan di VM2:
```
        ID NAME                     SALARY
---------- -------------------- ----------
      1001 Test Replikasi          9000000

ID_TRANSAKSI   ID_MERCHANT  NAMA_MERCHANT          JUMLAH STATUS
-------------- ------------ ---------------------- ------ -------
TRX001         MCH001       Warung Makan Pak Budi   50000 SUCCESS
```

---

## Cek Statistics di Web GUI

### VM1 — Extract Statistics

Akses `http://192.168.245.132:9001` → **EXTEMP** → **Statistics**

| Tabel | Inserts | Updates | Deletes |
|---|---|---|---|
| RAFI.EMPLOYEES | 1+ | 0 | 0 |
| RAFI.TRANSAKSI_QRIS | 1+ | 0 | 0 |

### VM1 — Distribution Statistics

Akses `http://192.168.245.132:9002` → **DISTSRC** → **Statistics**

| Metrik | Nilai |
|---|---|
| LCR Read from Trails | > 0 |
| LCR Sent | > 0 |

### VM2 — Replicat Statistics

Akses `http://192.168.245.133:9001` → **REPEMP** → **Statistics**

| Tabel | Inserts | Updates | Deletes |
|---|---|---|---|
| RAFI.EMPLOYEES | 1+ | 0 | 0 |
| RAFI.TRANSAKSI_QRIS | 1+ | 0 | 0 |

---

## Test Replikasi — Update dan Delete

```bash
# Di VM1 — Update data
sqlplus rafi/Rafi1234@ORCL << EOF
UPDATE RAFI.EMPLOYEES SET SALARY = 10000000 WHERE ID = 1001;
COMMIT;
EXIT
EOF

# Di VM2 — Verifikasi update terreplikasi
sqlplus rafi/Rafi1234@ORCL << EOF
SELECT * FROM RAFI.EMPLOYEES WHERE ID = 1001;
EXIT
EOF
```

---

## Ringkasan Status Komponen

**Checklist Replikasi Berhasil:**

**VM1 (Source):**
- ✅ Extract EXTEMP — Running
- ✅ Distribution DISTSRC — Running
- ✅ Trail file `et` — Ada data
- ✅ Statistics — LCR Read & Sent > 0

**VM2 (Target):**
- ✅ Receiver Service — Running
- ✅ Trail file `rt` — Ada data
- ✅ Replicat REPEMP — Running
- ✅ Statistics — Inserts > 0
- ✅ Data muncul di database target
