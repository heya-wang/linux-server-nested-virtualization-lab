# storage-server (TrueNAS iSCSI)

## Overview
Hostname: storage.gfn.internal  
IP: 192.168.10.12  
Mgmt URL: https://storage.gfn.internal  
Type: IP-SAN (iSCSI)  

## Resources

**CPU:** 2 vCPU  
**RAM:** 8 GB  
**Boot Disk:** 20 GB (da0)  
**Data Disks:** 3 × 1024 GiB  

## ZFS Pool

**Raw:** 3 TB  
**Usable:** ~2 TB  
**Disks:** Pool01, Pool02, Pool03，configured as RAID-Z (RAID 5 equivalent)  

## iSCSI

**Target:** target01  
**LUN:** 1500G  
**Block Size:** 4K  
**Portal:** 0.0.0.0:3260  
**Auth:** none  

## Services

iSCSI: enabled  
Web UI: enabled

## Network
**IP:** 192.168.10.12/24  
**Gateway:** 192.168.10.1

## DNS (BIND)
Add the following record to the zone file /etc/bind/db.gfn.internal on SRV1:
```dns
storage IN A 192.168.10.12

sudo systemctl reload named
```

## Proxmox Client
**Storage ID:** iscsi-storage  
**Portal IP:** 192.168.10.12  
**iSCSI Target:** target01 (auto-discovered)  
**Configuration:** Datacenter → Storage → Add → iSCSI → enter Portal IP → Query Target → select target01  
