# Zabbix Agent Installation
## Add Zabbix Repository
```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
apt update
```

##  Install Zabbix Agent
```bash
apt install zabbix-agent -y
```

## Configure Agent Connection
```bash
cat > /etc/zabbix/zabbix_agentd.conf << 'EOF'
Server=192.168.10.15
ServerActive=192.168.10.15
Hostname=srv1
EOF
```

## Start and Enable Agent
```bash
systemctl restart zabbix-agent
systemctl enable zabbix-agent
systemctl status zabbix-agent
```

## Add Host in Zabbix Web Interface
Open browser and navigate to http://192.168.10.15/zabbix/

Login with credentials (Admin / zabbix)

Go to Configuration → Hosts → Create Host

Enter the following settings:

Field	Value
Hostname	srv1
IP Address	192.168.10.11
Template	Linux by Zabbix agent
Group	Linux servers
Click Add to save
