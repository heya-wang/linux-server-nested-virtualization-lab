# SRV1 Configuration (Debian 13)

Created as VM on Proxmox:
> Hostname: srv1<br>
> IP: 192.168.10.11/24<br>
> Gateway: 192.168.10.1<br>
> Domain: gfn.internal

---

## Network

```bash
# Configure static network interface
cat > /etc/network/interfaces << 'EOF'
auto ens18
iface ens18 inet static
    address 192.168.10.11/24
    gateway 192.168.10.1
    dns-nameservers 127.0.0.1
EOF

# Install sudo and grant privileges to student user
apt update && apt install -y sudo
usermod -aG sudo student
```

---

## DNS (BIND9)

```bash
# Install BIND9
apt update && apt install -y bind9 bind9-utils

# Enable internal zones include
echo 'include "/etc/bind/named.conf.internal-zones";' >> /etc/bind/named.conf

# Main DNS options (forwarding + ACL)
cat > /etc/bind/named.conf.options << 'EOF'
acl internal-network {
    192.168.10.0/24;
};
options {
    directory "/var/cache/bind";

    # Upstream DNS servers
    forwarders {
        1.1.1.1;
        8.8.8.8;
    };

    # Allow queries only from local network
    allow-query { localhost; internal-network; };

    allow-transfer { none; };
    recursion yes;
    dnssec-validation no;

    # Disable IPv6
    listen-on-v6 { none; };
};
EOF

# Define zones
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

# Forward zone records
cat > /etc/bind/db.gfn.internal << 'EOF'
$TTL 604800
@ IN SOA srv1.gfn.internal. admin.gfn.internal. (1 604800 86400 2419200 604800)
@ IN NS srv1.gfn.internal.

# Hosts
proxmox IN A 192.168.10.10
router  IN A 192.168.10.1
CL01    IN A 192.168.10.50
srv1    IN A 192.168.10.11
storage IN A 192.168.10.12
srv2    IN A 192.168.10.13
EOF

# Reverse zone records
cat > /etc/bind/db.10.168.192 << 'EOF'
$TTL 604800
@ IN SOA srv1.gfn.internal. admin.gfn.internal. (1 604800 86400 2419200 604800)
@ IN NS srv1.gfn.internal.

# PTR records
13 IN PTR srv2.gfn.internal
11 IN PTR srv1.gfn.internal.
10 IN PTR proxmox.gfn.internal.
1  IN PTR router.gfn.internal.
50 IN PTR CL01.gfn.internal.
EOF

# Force IPv4 only
sed -i 's/^OPTIONS=.*/OPTIONS="-4"/' /etc/default/named

# Restart + enable service
systemctl restart bind9
systemctl enable bind9

# Set local resolver
echo "nameserver 127.0.0.1" > /etc/resolv.conf

# Test DNS resolution
dig proxmox.gfn.internal @192.168.10.11
```

---

## DHCP (KEA)

```bash
# Install KEA DHCP server
apt install -y kea

# Backup default config
mv /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.example

# Configure DHCP scope
cat > /etc/kea/kea-dhcp4.conf << 'EOF'
{
    "Dhcp4": {

        # Interface binding
        "interfaces-config": {
            "interfaces": [ "ens18" ]
        },

        # Control socket
        "control-socket": {
            "socket-type": "unix",
            "socket-name": "/run/kea/kea4-ctrl-socket"
        },

        # Lease database
        "lease-database": {
            "type": "memfile",
            "lfc-interval": 3600
        },

        # Lease timers
        "valid-lifetime": 600,
        "max-valid-lifetime": 7200,

        # Network definition
        "subnet4": [
            {
                "id": 1,
                "subnet": "192.168.10.0/24",

                # IP pool
                "pools": [
                    {
                        "pool": "192.168.10.50 - 192.168.10.200"
                    }
                ],

                # DHCP options
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

# Restart + enable DHCP
systemctl restart kea-dhcp4-server
systemctl enable kea-dhcp4-server
```

---

## SSH Server (SRV1)

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

## LDAP Server (srv1)

```bash
# Install LDAP
sudo apt update
sudo apt install slapd ldap-utils -y

# Configure LDAP
sudo dpkg-reconfigure slapd
# DNS domain: gfn.internal
# Organization: gfn.internal
# Backend: MDB

# Verify
sudo systemctl status slapd
sudo ss -tlnp | grep 389

ldapsearch -x -b "dc=gfn,dc=internal" \
-D "cn=admin,dc=gfn,dc=internal" -W

# Install LAM (Web UI)
sudo apt install apache2 php php-ldap php-mbstring php-xml php-json -y
sudo apt install ldap-account-manager -y

sudo systemctl enable apache2
sudo systemctl restart apache2
```

Web:
http://192.168.10.11/lam

Login:
lam / lam

Config:

* Server: ldap://srv1.gfn.internal
* Base DN: dc=gfn,dc=internal

Users and groups are managed via LAM

---





