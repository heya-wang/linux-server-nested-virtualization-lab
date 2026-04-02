# Router Installation (Linux Gateway)

## Hyper-V VM Creation

Name: Router
RAM: 1024 MB, dynamic 
2 network cards  
Debian 13

---

## Network Configuration

Login as root:

```bash
cat > /etc/network/interfaces << EOF
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
    address 192.168.10.1
    netmask 255.255.255.0
EOF

sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
```
---

## NAT rules
```bash
apt install iptables -y
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
apt install iptables-persistent -y
reboot
```
---
