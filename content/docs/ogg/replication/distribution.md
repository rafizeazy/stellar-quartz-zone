---
title: Distribution Path (VM1)
linkTitle: Distribution Path
weight: 52
---

Distribution Path bertugas **mengirimkan trail file** dari VM1 (Source) ke VM2 (Target) melalui jaringan menggunakan protokol OGG.

---

## Buat Distribution Path di VM1

### Akses Distribution Service

```
http://192.168.245.132:9002
```

Login: `ggadmin` / `@Rafi1234`

### Langkah Membuat Distribution Path

1. Klik **+** untuk tambah path baru
2. Isi form:

| Field | Nilai |
|---|---|
| Path Name | `DISTSRC` |
| Description | `Distribution dari VM1 ke VM2` |
| Source | Pilih **EXTEMP** dari dropdown |
| Trail Name | `et` |
| Target | `ogg` |
| Target Host | `192.168.245.133` |
| Port Number | `9003` |
| Trail Name (Target) | `rt` |
| Target Encryption Algorithm | `NONE` |
| Trail Size (MB) | `500` |
| Target Type | `Receiver Service` |
| Begin | `Now` |
| Auto Restart | `Yes` |

> **Penting — Pilih Source dari Dropdown:** Pastikan field **Source** dipilih dari **dropdown EXTEMP**, bukan diketik manual. Ini memastikan Generated Source URI mengandung parameter `extract=EXTEMP`:
> ```
> trail://192.168.245.132:9002/services/v2/sources?trail=et&extract=EXTEMP
> ```
> Jika Source URI tidak mengandung `extract=EXTEMP`, Distribution tidak akan bisa membaca trail file.

3. Klik **Create and Run**

---

## Verifikasi Distribution Path

### Cek Path Information

Klik **DISTSRC** → tab **Path Information**. Pastikan:

| Field | Nilai yang Diharapkan |
|---|---|
| Status | running |
| Source | `trail://192.168.245.132:9002/services/v2/sources?trail=et&extract=EXTEMP` |
| Target | `ogg://192.168.245.133:9003/services/v2/targets?trail=rt` |
| DB Name | Nama database (bukan UNKNOWN) |
| Extract | EXTEMP (bukan UNKNOWN) |

> **Jika DB Name dan Extract UNKNOWN:** Artinya Source URI tidak terhubung ke Extract EXTEMP. Delete dan buat ulang Distribution Path, pastikan **Source dipilih dari dropdown EXTEMP**.

### Cek Statistics

Klik tab **Statistics**. Setelah ada transaksi, nilai **LCR Read from Trails** dan **LCR Sent** akan berubah dari 0.

---

## Verifikasi Trail File di VM2

```bash
# Cek trail file rt sudah ada di VM2
ls -la /u01/app/ogg_depot/oggtarget/var/lib/data/rt*
```

Jika trail file ada dan ukurannya > 0 bytes, Distribution berhasil mengirim data.

---

## Troubleshooting Distribution Path

### LCR Read from Trails tetap 0

1. Pastikan Source URI mengandung `extract=EXTEMP`
2. Pastikan parameter EXTEMP menggunakan `EXTTRAIL et` (bukan path lengkap)
3. Restart Extract EXTEMP lalu cek kembali

### Error: JSON element does not match any schemas

Format Target URI salah. Gunakan field **Target Host**, **Port Number**, dan **Trail Name** yang tersedia, jangan edit URI manual.

### Trail file di VM2 kosong (0 bytes)

Cek koneksi dari VM1 ke port 9003 VM2:
```bash
curl -v http://192.168.245.133:9003
```
