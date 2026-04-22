---
title: Gambaran Umum
linkTitle: Gambaran Umum
weight: 21
---

## Tujuan Dokumentasi

Dokumentasi ini menjelaskan langkah-langkah lengkap untuk menginstal dan mengkonfigurasi **Oracle GoldenGate 21c** dalam arsitektur **Microservices** untuk replikasi data real-time dari database Oracle source ke Oracle target menggunakan dua Virtual Machine (VM).

---

## Skenario Replicat

Replikasi yang diimplementasikan adalah **Unidirectional Replication** (satu arah), yaitu perubahan data di VM1 (Source) akan secara otomatis direplikasi ke VM2 (Target) secara real-time.

```
VM1 (Source)  ──────────────────►  VM2 (Target)
192.168.245.132                    192.168.245.133
```

### Tabel yang Direplikasi

| Schema | Tabel | Keterangan |
|---|---|---|
| RAFI | EMPLOYEES | Data karyawan |
| RAFI | TRANSAKSI_QRIS | Data transaksi QRIS |

---

## Workflow OGG

```
1. Oracle DB (Source)
   └── Redo Log
       └── Extract (EXTEMP)        ← Membaca perubahan dari redo log
           └── Trail File (et)     ← Menyimpan perubahan sementara
               └── Distribution (DISTSRC)  ← Mengirim ke target
                   └── [NETWORK]
                       └── Receiver Service (Port 9003)
                           └── Trail File (rt)  ← Diterima di target
                               └── Replicat (REPEMP)  ← Apply ke DB target
                                   └── Oracle DB (Target)
```

---

## Versi Software

| Software | Versi |
|---|---|
| Oracle GoldenGate | 21.3.0.0.0 |
| Oracle Database | 19c (19.3.0.0.0) |
| Oracle Linux | 7 (UEK) |
| Java (JDK) | Bundled dengan OGG |

---

## Link Download

| File | Keterangan | Link |
|---|---|---|
| OGG 21c untuk Oracle | Installer OGG Microservices | [Download di Oracle Software Delivery](https://edelivery.oracle.com/) |
| Oracle Database 19c | Installer Oracle DB | [Download di Oracle Technology Network](https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html) |
| Oracle Linux 7 | OS yang digunakan | [Download Oracle Linux](https://yum.oracle.com/oracle-linux-isos.html) |

> **Perhatian:** File installer OGG memerlukan akun Oracle (gratis). Daftarkan akun di [oracle.com](https://profile.oracle.com/myprofile/account/create-account.jspx).
