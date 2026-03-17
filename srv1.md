# SRV1 - DNS Server Configuration

## Virtual Machine Settings
- **Hostname:** srv1
- **OS:** Debian 13
- **IP:** 192.168.10.11/24
- **Gateway:** 192.168.10.1
- **DNS:** 127.0.0.1
- **Domain:** gfn.internal

---

## Complete Setup

```bash
# Network Configuration
cat > /etc/network/interfaces << 'EOF'
auto ens18
iface ens18 inet static
    address 192.168.10.11/24
    gateway 192.168.10.1
    dns-nameservers 127.0.0.1
EOF

# Install BIND9
apt update && apt install -y bind9 bind9-utils

# Configure named.conf
echo 'include "/etc/bind/named.conf.internal-zones";' >> /etc/bind/named.conf

# Configure named.conf.options
cat > /etc/bind/named.conf.options << 'EOF'
acl internal-network {
    192.168.10.0/24;
};
options {
    directory "/var/cache/bind";
    forwarders {
        1.1.1.1;
        8.8.8.8;
    };
    allow-query { localhost; internal-network; };
    allow-transfer { none; };
    recursion yes;
    dnssec-validation no;
    listen-on-v6 { none; };
};
EOF

# Configure internal zones
cat > /etc/bind/named.conf.internal-zones << 'EOF'
zone "gfn.internal" {
    type master;
    file "/etc/bind/db.gfn.internal";
};
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.168.192";
};
EOF

# Configure forward zone
cat > /etc/bind/db.gfn.internal << 'EOF'
$TTL 604800
@ IN SOA srv1.gfn.internal. admin.gfn.internal. (1 604800 86400 2419200 604800)
@ IN NS srv1.gfn.internal.
srv1    IN A 192.168.10.11
proxmox IN A 192.168.10.10
router  IN A 192.168.10.1
CL01    IN A 192.168.10.50
EOF

# Configure reverse zone
cat > /etc/bind/db.10.168.192 << 'EOF'
$TTL 604800
@ IN SOA srv1.gfn.internal. admin.gfn.internal. (1 604800 86400 2419200 604800)
@ IN NS srv1.gfn.internal.
11 IN PTR srv1.gfn.internal.
10 IN PTR proxmox.gfn.internal.
1  IN PTR router.gfn.internal.
50 IN PTR CL01.gfn.internal.
EOF

# Disable IPv6 for bind
sed -i 's/^OPTIONS=.*/OPTIONS="-4"/' /etc/default/named

# Restart and enable
systemctl restart bind9
systemctl enable bind9

# Set local DNS
echo "nameserver 127.0.0.1" > /etc/resolv.conf

# Test DNS
dig proxmox.gfn.internal @192.168.10.11
```
# SRV1 - DHCP Server Configuration

## Complete Setup

\`\`\`bash
# Install KEA DHCP Server
apt update && apt install -y kea

# Backup default configuration
mv /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.example

# Configure KEA DHCP
cat > /etc/kea/kea-dhcp4.conf << 'EOF'
{
    "Dhcp4": {
        "interfaces-config": {
            "interfaces": [ "ens18" ]
        },
        "control-socket": {
            "socket-type": "unix",
            "socket-name": "/run/kea/kea4-ctrl-socket"
        },
        "lease-database": {
            "type": "memfile",
            "lfc-interval": 3600
        },
        "valid-lifetime": 600,
        "max-valid-lifetime": 7200,
        "subnet4": [
            {
                "id": 1,
                "subnet": "192.168.10.0/24",
                "pools": [
                    {
                        "pool": "192.168.10.100 - 192.168.10.200"
                    }
                ],
                "option-data": [
                    {
                        "name": "routers",
                        "data": "192.168.10.1"
                    },
                    {
                        "name": "domain-name-servers",
                        "data": "192.168.10.11"
                    },
                    {
                        "name": "domain-name",
                        "data": "gfn.internal"
                    }
                ]
            }
        ]
    }
}
EOF

# Restart and enable DHCP server
systemctl restart kea-dhcp4-server
systemctl enable kea-dhcp4-server
systemctl status kea-dhcp4-server --no-pager
\`\`\`

## Test DHCP Client (on CL01)
\`\`\`bash
# On CL01, switch to DHCP
sudo dhclient -v
ip addr show
\`\`\`
