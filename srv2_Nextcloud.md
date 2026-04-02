# Nextcloud Installation on SRV2

## Prerequisites

Verify Apache2 is installed and running:

```bash
sudo systemctl status apache2

# If Apache2 is not installed:
sudo apt update && sudo apt install apache2 -y
```
---

## PHP 8.4 Installation
Install PHP 8.4 and required extensions
```bash
sudo apt update
sudo apt install -y php php-curl php-cli php-mysql php-gd php-common php-xml php-json php-intl php-pear php-imagick php-dev php-common php-mbstring php-zip php-soap php-bz2 php-bcmath php-gmp php-apcu libmagickcore-dev<img width="912" height="99" alt="image" src="https://github.com/user-attachments/assets/3597f742-d3a2-4390-beca-c5ff91050aab" />

# Verify PHP version and loaded modules
php --version
php -m

```
---

## Edit PHP configuration

```bash
sudo nano /etc/php/8.4/apache2/php.ini
```

Create the PHP configuration file with the following initial content:

```ini
; /etc/php/8.4/apache2/php.ini

; Timezone
date.timezone = Europe/Berlin

; Resource Limits
memory_limit = 512M
upload_max_filesize = 500M
post_max_size = 600M
max_execution_time = 300

; File Uploads
file_uploads = On
allow_url_fopen = On

; Error Handling
display_errors = Off
output_buffering = Off

; OPcache
zend_extension = opcache

[opcache]
opcache.enable = 1
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 10000
opcache.memory_consumption = 128
opcache.save_comments = 1
opcache.revalidate_freq = 1
```

Restart Apache to apply changes
```bash
sudo systemctl restart apache2
```

---

## MariaDB Installation
```bash
#Install MariaDB server
sudo apt install -y mariadb-server

# Verify service status
sudo systemctl status mariadb

# Secure MariaDB installation
sudo mariadb-secure-installation

#Follow the prompts:
# Press Enter for current password
# n for switching to unix_socket authentication
# y to set root password
# Enter password:
# y to remove anonymous users
# y to disallow root login remotely
# y to remove test database
# y to reload privilege tables

```
---

## Database setup
```bash
mariadb -u root -p <<EOF
CREATE DATABASE nextcloud_db;
CREATE USER nextclouduser@localhost IDENTIFIED BY '<Password>';
GRANT ALL PRIVILEGES ON nextcloud_db.* TO nextclouduser@localhost;
FLUSH PRIVILEGES;
EXIT;
EOF

SHOW GRANTS FOR nextclouduser@localhost
```
---

## Download Nextcloud
```bash
# Install required tools
sudo apt install -y curl unzip

# Navigate to web root and download Nextcloud
cd /var/www/
sudo curl -o nextcloud.zip https://download.nextcloud.com/server/releases/latest.zip

# Extract and set permissions
sudo unzip nextcloud.zip
sudo chown -R www-data:www-data nextcloud

```
---

## Apache Virtual Host Configuration
```bash
# Create virtual host configuration
sudo tee /etc/apache2/sites-available/nextcloud.gfn.internal.conf > /dev/null << 'EOF'
<VirtualHost *:80>
  ServerName nextcloud.gfn.internal
  DocumentRoot /var/www/nextcloud/
  <Directory /var/www/nextcloud/>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews
        <IfModule mod_dav.c>
            Dav off
        </IfModule>
  </Directory>
</VirtualHost>
EOF

# Test configuration and enable site
sudo apachectl configtest
sudo a2ensite nextcloud.gfn.internal.conf
sudo systemctl reload apache2

```
---

## DNS Configuration on srv1
On SRV1, edit the BIND zone file:
```bash
sudo nano /etc/bind/db.gfn.internal
# Add the CNAME record:
nextcloud		IN	CNAME		srv2

```
---

## Nextcloud Setup
Open browser and navigate to http://nextcloud.gfn.internal.
Create an admin user with username and password.
Leave data directory as default.
Scroll to database section and enter username, password database name and database host.
Click "Install".
Confirm installation of recommended Nextcloud apps.
