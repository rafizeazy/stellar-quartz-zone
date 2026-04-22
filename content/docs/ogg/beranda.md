---
title: Beranda
linkTitle: Beranda
weight: 10
---

## Apa itu Oracle GoldenGate?

**Oracle GoldenGate (OGG)** adalah solusi replikasi data real-time dari Oracle yang memungkinkan transfer data antar database dengan latensi rendah. OGG bekerja dengan membaca **redo log** dari database sumber dan mengaplikasikan perubahan ke database target secara real-time.

OGG 21c menggunakan arsitektur **Microservices** berbasis REST API dengan antarmuka web (Web GUI) untuk manajemen proses replikasi.

---

## Arsitektur Replicat

```
┌─────────────────────────────────────────────────────────────────┐
│                        ALUR REPLIKASI OGG                       │
├──────────────────┬──────────────────┬───────────────────────────┤
│   VM1 (Source)   │    NETWORK       │      VM2 (Target)         │
│                  │                  │                           │
│  Oracle DB       │                  │   Oracle DB               │
│  (ORCL)          │                  │   (ORCL)                  │
│     │            │                  │      ▲                    │
│     │ Redo Log   │                  │      │                    │
│     ▼            │                  │      │                    │
│  Extract         │                  │   Replicat               │
│  (EXTEMP)        │                  │   (REPEMP)               │
│     │            │                  │      ▲                    │
│     │ Trail et   │                  │      │ Trail rt           │
│     ▼            │                  │      │                    │
│  Distribution ───┼──────────────────┼───► Receiver             │
│  (DISTSRC)       │   Port 9003      │      Service             │
│  Port 9002       │                  │                           │
└──────────────────┴──────────────────┴───────────────────────────┘
```

---

## Komponen Utama

| Komponen | Port | Fungsi |
|---|---|---|
| **Service Manager** | 55000 | Manajemen pusat semua service OGG |
| **Administration Service** | 9001 | Kelola Extract dan Replicat |
| **Distribution Service** | 9002 | Kirim trail file ke target |
| **Receiver Service** | 9003 | Terima trail file dari source |
| **Performance Metrics** | 9004 | Monitor performa replikasi |

---

## Environment

| Item | VM1 (Source) | VM2 (Target) |
|---|---|---|
| **Hostname** | oggrafi-src | oggrafi-tgt |
| **IP Address** | 192.168.245.132 | 192.168.245.133 |
| **OS** | Oracle Linux 7 | Oracle Linux 7 |
| **Oracle DB** | 19c (ORCL) | 19c (ORCL) |
| **OGG Version** | 21.3.0.0.0 | 21.3.0.0.0 |
| **OGG Home** | /u01/app/ogg | /u01/app/ogg |
| **Deployment** | oggsource | oggtarget |

---

## Cara akses GUI

### VM1 (Source)

| Service | URL |
|---|---|
| Service Manager | http://192.168.245.132:55000 |
| Administration Service | http://192.168.245.132:9001 |
| Distribution Service | http://192.168.245.132:9002 |
| Receiver Service | http://192.168.245.132:9003 |

### VM2 (Target)

| Service | URL |
|---|---|
| Service Manager | http://192.168.245.133:55000 |
| Administration Service | http://192.168.245.133:9001 |
| Distribution Service | http://192.168.245.133:9002 |
| Receiver Service | http://192.168.245.133:9003 |

> **Kredensial Login:** Username: `ggadmin` / Password: `@Rafi1234`

---

{{< button relref="/docs/ogg/getting-started/" >}}Mulai →{{< /button >}}
