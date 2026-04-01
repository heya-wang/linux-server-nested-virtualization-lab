# SRV2 Configuration (Debian 13)

Created as VM on Proxmox:
> Hostname: srv2<br>
> IP: 192.168.10.13/24<br>
> Gateway: 192.168.10.1<br>
> Domain: gfn.internal

---

## Network and System Setup

```bash
# Configure static network interface
cat > /etc/network/interfaces << 'EOF'
auto ens18
iface ens18 inet static
    address 192.168.10.13/24
    gateway 192.168.10.1
    dns-nameservers 127.0.0.1
EOF

# Apply network configuration
systemctl restart networking

# Install sudo and grant privileges to student user
apt update && apt install -y sudo
usermod -aG sudo student

```
---

## DNS Configuration (SRV1)
```bash
# Edit zone file
vim /etc/bind/db.gfn.internal

# Add record
srv2    IN    A    192.168.10.13

# Reload DNS
systemctl reload named

```
---

## SSH Server 
```bash
# Install OpenSSH server 
sudo apt install -y openssh-server

# Edit SSH server configuration
sudo nano /etc/ssh/sshd_config

# Ensure the following options are enabled:
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# Reload SSH service to apply changes
sudo systemctl reload sshd

# Disable Password Authentication (Server Hardening)
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config

# Change this setting:
PasswordAuthentication no

# Reload SSH service
sudo systemctl reload sshd
```

---

## SRV2 Backup over iSCSI (target02)
### Create a new iSCSI Target:

**Target name:** target02  
**Size:** 200 GB  
**Authentication:** None  
**Access:** Authorized for SRV2

### iSCSI Client Setup on SRV2
```bash
# Switch to root
su -

# Install iSCSI tools
apt update && apt install -y open-iscsi rsync

# Discover available targets
iscsiadm -m discovery -t sendtargets -p 192.168.10.12

# Connect to target02
iscsiadm -m node -T iqn.2005-10.org.freenas.ctl:target02 -p 192.168.10.12 -l

# Verify the new block device
lsblk

# Partition the device
cfdisk /dev/sdb
# Inside fdisk: n (new) → p (primary) → Enter (default start) → Enter (default end) → w (write)

# Format the partition
mkfs.ext4 /dev/sdb1

# Create mount point and mount
mkdir -p /mnt/backup
mount /dev/sdb1 /mnt/backup

# Verify mount
df -h /mnt/backup

# Unmount and disconnect
umount /mnt/backup
iscsiadm -m node -T iqn.2005-10.org.freenas.ctl:target02 -p 192.168.10.12 -u
```

### Backup Script

```bash
# Create /usr/local/bin/backup.sh:
nano /usr/local/bin/backup.sh

#!/bin/bash
# Variablen
ISCSI_TARGET="iqn.2005-10.org.freenas.ctl:target02"  # iSCSI-Target-Name
ISCSI_IP="192.168.10.12"  # IP-Adresse des iSCSI-Servers
MOUNT_POINT="/mnt/backup"
BACKUP_SOURCE1="/var/www"
BACKUP_SOURCE2="/etc"
DATE=$(date +"%Y-%m-%d")
BACKUP_DIR="$MOUNT_POINT/$DATE"

# iSCSI-Target mounten
sudo iscsiadm -m node -T $ISCSI_TARGET -p $ISCSI_IP -l

# Warten, bis das iSCSI-Target verfügbar ist
sleep 5

# Verzeichnis erstellen, wenn nicht vorhanden
if [ ! -d "$MOUNT_POINT" ]; then
    sudo mkdir -p $MOUNT_POINT
fi

# iSCSI-Target mounten
sudo mount /dev/disk/by-path/ip-$ISCSI_IP:3260-iscsi-$ISCSI_TARGET-lun-0-part1 $MOUNT_POINT

# Backup-Verzeichnis erstellen
sudo mkdir -p $BACKUP_DIR

# Backup erstellen
sudo rsync -a $BACKUP_SOURCE1 $BACKUP_DIR
sudo rsync -a $BACKUP_SOURCE2 $BACKUP_DIR

# iSCSI-Target aushängen
sudo umount $MOUNT_POINT

# iSCSI-Target logout
sudo iscsiadm -m node -T $ISCSI_TARGET -p $ISCSI_IP -u


# Make script executable
chmod +x /usr/local/bin/backup.sh

```

### Test and Verification
```bash
# Test the script manually
/usr/local/bin/backup.sh
# Note: /var/www does not exist yet, ignore the error

# Verify backup data
iscsiadm -m node -T iqn.2005-10.org.freenas.ctl:target02 -p 192.168.10.12 -l
mount /dev/sdb1 /mnt/backup
ls -lha /mnt/backup

# Clean up
umount /mnt/backup
iscsiadm -m node -T iqn.2005-10.org.freenas.ctl:target02 -p 192.168.10.12 -u

# Edit root's crontab
crontab -e

# Add the following line:
0 1 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```
---

