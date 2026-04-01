# Zabbix Network Monitoring Server Installation
## Overview
Create a Debian virtual machine on Hyper-V with:

IP Address: 192.168.10.15

Gateway: 192.168.10.1

DNS Server: 192.168.10.11  8.8.8.8 1.1.1.1

---

## Zabbix Server installation
```bash
# Add Zabbix Repository
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
sudo apt update

# Install Zabbix Server
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y

# Configure PHP Timezone
cat > /etc/php/8.4/apache2/php.ini << 'EOF'
; PHP configuration file
date.timezone = Europe/Berlin
EOF
```


## Install and Configure MariaDB
```bash
# Install MariaDB
apt install mariadb-server -y
sudo systemctl start mariadb

# Creat database
sudo mysql -u root -p
 CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;  二进制排序规则（区分大小写）
CREATE USER zabbix@localhost;
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost  IDENTIFIED BY 'password'
MariaDB [(none)]> FLUSH PRIVILEGES; 
MariaDB [(none)]> EXIT; 

# Configure Zabbix server database connection
cat > /etc/zabbix/zabbix_server.conf << 'EOF'
DBName=zabbix
DBUser=zabbix
DBPassword=<password>
EOF

#Import database
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
systemctl restart apache2

```
---
## Start Services
```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
systemctl status zabbix-server
systemctl status zabbix-agent
```
---


## Access Web Interface
On cl01 open http://192.168.10.11/zabbix and complete the setup wizard:

Database: zabbix / zabbix / <password>

Login: Admin / zabbix
