---
title: Konfigurasi IP Statis
linkTitle: Konfigurasi IP Statis
weight: 33
---

> **Penting:** IP statis **wajib** dikonfigurasi agar IP VM tidak berubah saat ganti jaringan/hotspot. Jika IP berubah, konfigurasi TNS dan OGG Distribution Path harus diupdate ulang.

## VM1 (Source) — Login sebagai root

```bash
su -

# Set IP statis untuk interface ens33
nmcli con mod ens33 ipv4.addresses 192.168.245.132/24
nmcli con mod ens33 ipv4.gateway 192.168.245.2
nmcli con mod ens33 ipv4.dns 8.8.8.8
nmcli con mod ens33 ipv4.method manual
nmcli con up ens33

# Verifikasi
ip addr show ens33 | grep "inet "
```

Output yang diharapkan:
```
inet 192.168.245.132/24 brd 192.168.245.255 scope global noprefixroute ens33
```

---

## VM2 (Target) — Login sebagai root

```bash
su -

# Set IP statis untuk interface ens33
nmcli con mod ens33 ipv4.addresses 192.168.245.133/24
nmcli con mod ens33 ipv4.gateway 192.168.245.2
nmcli con mod ens33 ipv4.dns 8.8.8.8
nmcli con mod ens33 ipv4.method manual
nmcli con up ens33

# Verifikasi
ip addr show ens33 | grep "inet "
```

---

## Update tnsnames.ora Setelah IP Berubah

Jika IP berubah, update file tnsnames.ora di **kedua VM**:

### VM1 (Source)

```bash
cat > /u01/app/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora << 'EOF'
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.245.132)(PORT = 1521))
    (CONNECT_DATA =
      (SERVICE_NAME = ORCL)
    )
  )
EOF

# Verifikasi
tnsping ORCL
```

### VM2 (Target)

```bash
cat > /u01/app/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora << 'EOF'
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.245.133)(PORT = 1521))
    (CONNECT_DATA =
      (SERVICE_NAME = ORCL)
    )
  )
EOF

# Verifikasi
tnsping ORCL
```

> **Tips:** Jika `tnsping ORCL` gagal dengan error `TNS-03505: Failed to resolve name`, pastikan tidak ada spasi di awal baris `ORCL =` pada file tnsnames.ora.
