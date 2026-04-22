---
title: Konfigurasi Credential OGG
linkTitle: Credential OGG
weight: 43
---

Credential OGG digunakan oleh Extract dan Replicat untuk login ke Oracle Database.

---

## Buat Credential di VM1 (Source)

### Akses Administration Service VM1

Buka browser dan akses:
```
http://192.168.245.132:9001
```

Login: `ggadmin` / `@Rafi1234`

### Tambah Credential

1. Klik menu **≡ (hamburger)** → **Configuration**
2. Pilih tab **Credentials**
3. Klik tombol **+**
4. Isi form:

| Field | Nilai |
|---|---|
| Credential Domain | `OracleGoldenGate` |
| Credential Alias | `srcdb` |
| User ID | `ggadmin@ORCL` |
| Password | `Rafi1234` |

5. Klik **Submit**
6. Klik icon **🔌** untuk test koneksi

> **Berhasil:** Jika test koneksi berhasil, akan muncul informasi **Trandata** dan **Checkpoint** di bawah credential.

---

## Buat Credential di VM2 (Target)

### Akses Administration Service VM2

```
http://192.168.245.133:9001
```

Login: `ggadmin` / `@Rafi1234`

### Tambah Credential

1. Klik menu **≡** → **Configuration** → **Credentials**
2. Klik **+**
3. Isi form:

| Field | Nilai |
|---|---|
| Credential Domain | `OracleGoldenGate` |
| Credential Alias | `tgtdb` |
| User ID | `ggadmin@ORCL` |
| Password | `Rafi1234` |

4. Klik **Submit**
5. Test koneksi dengan klik icon **🔌**

---

## Daftarkan Checkpoint Table di VM2

Setelah credential `tgtdb` terhubung:

1. Scroll ke bawah ke bagian **Checkpoint**
2. Klik **+**
3. Isi: `GGADMIN.CHECKPOINTTABLE`
4. Klik **Submit**

---

## Manajemen User OGG

User untuk login ke Web GUI OGG dikelola di Service Manager.

### Tambah User Baru via adminclient

```bash
# Login sebagai oracle di VM1
/u01/app/ogg/bin/adminclient
```

Setelah masuk adminclient:
```
connect http://192.168.245.132:55000 deployment oggsource as ggadmin password @Rafi1234
create user namauser password Password123# role administrator
info security users *
exit
```

### Role yang Tersedia

| Role | Keterangan |
|---|---|
| `Administrator` | Akses penuh ke semua fitur |
| `Operator` | Bisa start/stop process |
| `Viewer` | Hanya bisa melihat status |
