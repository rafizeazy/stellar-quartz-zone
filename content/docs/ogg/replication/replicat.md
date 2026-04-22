---
title: Replicat (Target — VM2)
linkTitle: Replicat (VM2)
weight: 53
---

Replicat adalah komponen OGG yang bertugas **membaca trail file** yang dikirim dari VM1 dan **mengaplikasikan perubahan** ke database target di VM2.

---

## Buat Replicat di VM2

### Akses Administration Service VM2

```
http://192.168.245.133:9001
```

Login: `ggadmin` / `@Rafi1234`

### Langkah Membuat Replicat

1. Di halaman **Overview**, klik **+** di bagian **Replicats**
2. Pilih type **Nonintegrated Replicat** → klik **Next**
3. Isi form **Basic Information**:

| Field | Nilai |
|---|---|
| Process Name | `REPEMP` |
| Description | `Replicat target dari VM1` |
| Intent | `Unidirectional` |
| Credential Domain | `OracleGoldenGate` |
| Credential Alias | `tgtdb` |
| Source | `Trail` |
| Trail Name | `rt` |
| Trail Subdirectory | *(kosongkan)* |
| Begin | `Now` |
| Checkpoint Table | `GGADMIN.CHECKPOINTTABLE` |

> **Penting — Trail Name:** Trail Name harus `rt` (bukan `et`). Trail `rt` adalah trail yang dikirim oleh Distribution Path dari VM1 ke VM2.

4. Klik **Next**

5. Isi **Parameter File**:

```
REPLICAT REPEMP
USERIDALIAS tgtdb DOMAIN OracleGoldenGate
ASSUMETARGETDEFS
MAP RAFI.EMPLOYEES, TARGET RAFI.EMPLOYEES;
MAP RAFI.TRANSAKSI_QRIS, TARGET RAFI.TRANSAKSI_QRIS;
```

**Penjelasan parameter:**

| Parameter | Keterangan |
|---|---|
| `USERIDALIAS tgtdb` | Gunakan credential alias `tgtdb` |
| `ASSUMETARGETDEFS` | Asumsikan struktur tabel source dan target sama |
| `MAP ... TARGET ...` | Mapping tabel source ke tabel target |

6. Klik **Create and Run**

---

## Verifikasi Replicat

### Cek Status

Pastikan REPEMP berstatus **Running**.

### Cek Checkpoint

Klik **REPEMP** → tab **Checkpoint**. Pastikan **Sequence** dan **Offset** berubah setelah ada transaksi di VM1.

### Cek Statistics

Klik tab **Statistics**. Setelah ada transaksi di VM1, nilai **Inserts/Updates/Deletes** akan bertambah.

---

## Troubleshooting Replicat

### Statistics tidak berubah, Inserts tetap 0

1. Pastikan trail name `rt` sudah benar
2. Pastikan trail file `rt` di VM2 tidak kosong:
    ```bash
    ls -la /u01/app/ogg_depot/oggtarget/var/lib/data/rt*
    ```
3. Cek Checkpoint — pastikan Sequence dan Offset berubah

### Error: ORA-00942 table or view does not exist

Tabel di database target belum dibuat. Buat tabel dengan struktur yang sama persis dengan source.

### Error: OGG-08110 Login failed

Credential `tgtdb` tidak bisa konek ke database. Pastikan:

1. tnsnames.ora di VM2 sudah benar (IP = 192.168.245.133)
2. Listener Oracle berjalan: `lsnrctl status`
3. Service ORCL terdaftar di listener
