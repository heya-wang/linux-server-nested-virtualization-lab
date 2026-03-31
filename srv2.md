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

## DNS Configuration (SRV1)
```bash
# Edit zone file
vim /etc/bind/db.gfn.internal

# Add record
srv2    IN    A    192.168.10.13

# Reload DNS
systemctl reload named

```
