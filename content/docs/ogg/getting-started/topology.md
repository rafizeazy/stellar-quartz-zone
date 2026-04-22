---
title: Topologi
linkTitle: Topologi
weight: 23
---

## Diagram Topologi

<div style="text-align: center;">
  <img src="../assets/images/topology.png" alt="Diagram Topologi OGG" style="max-width: 100%; border-radius: 8px; border: 1px solid #ddd;">
  <p><em>placeholder: topology.png — Diagram topologi jaringan VM1 dan VM2</em></p>
</div>

---

## Detail Jaringan

### VM1 (Source) — oggrafi-src

| Interface | IP Address | Subnet | Keterangan |
|---|---|---|---|
| ens33 | 192.168.245.132 | /24 | Network utama (NAT/Bridged) |
| ens34 | 192.168.197.101 | /24 | Network internal (Host-only) |

### VM2 (Target) — oggrafi-tgt

| Interface | IP Address | Subnet | Keterangan |
|---|---|---|---|
| ens33 | 192.168.245.133 | /24 | Network utama (NAT/Bridged) |
| ens34 | 192.168.197.102 | /24 | Network internal (Host-only) |

---

## Alur Komunikasi Antar VM

```
VM1 (192.168.245.132)                    VM2 (192.168.245.133)
┌─────────────────────┐                  ┌─────────────────────┐
│                     │                  │                     │
│  Distribution Svc   │  ─── 9003 ──►   │  Receiver Service   │
│  (Port 9002)        │                  │  (Port 9003)        │
│                     │                  │                     │
│  Oracle DB (1521)   │                  │  Oracle DB (1521)   │
│                     │                  │                     │
└─────────────────────┘                  └─────────────────────┘
```

---

## Direktori Penting

### VM1 (Source)

| Direktori | Keterangan |
|---|---|
| `/u01/app/ogg` | OGG Software Home |
| `/u01/app/ogg_deployments` | Service Manager Deployment Home |
| `/u01/app/ogg_depot/oggsource` | Deployment oggsource |
| `/u01/app/ogg_depot/oggsource/var/lib/data/dirdat` | Lokasi trail file (et) |
| `/u01/app/oraInventory` | Oracle Inventory |

### VM2 (Target)

| Direktori | Keterangan |
|---|---|
| `/u01/app/ogg` | OGG Software Home |
| `/u01/app/ogg_deployments` | Service Manager Deployment Home |
| `/u01/app/ogg_depot/oggtarget` | Deployment oggtarget |
| `/u01/app/ogg_depot/oggtarget/var/lib/data` | Lokasi trail file (rt) |
| `/u01/app/oraInventory` | Oracle Inventory |
