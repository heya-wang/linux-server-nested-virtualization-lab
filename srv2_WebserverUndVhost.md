# Apache2 Virtual Host Setup

## Install Apache2

```bash
sudo apt install apache2 -y

# Check service
sudo systemctl status apache2
ss -tapn

# Create document root and index.html
sudo mkdir -p /var/www/gfnkurs
sudo tee /var/www/gfnkurs/index.html > /dev/null << 'EOF'

<!DOCTYPE html>
<html lang="de">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GFN Kursseite</title>
  </head>
  <body>
    <h1>Herzlich Willkommen!</h1>
    <p>Dies ist eine Dummyseite, um die Funktion des Apache Webservers zu demonstrieren.</p>
  </body>
</html>
EOF
```

## Create virtual host configuration
```bash
sudo tee /etc/apache2/sites-available/gfnkurs.gfn.internal.conf > /dev/null << 'EOF'

<VirtualHost *:80>
  ServerName gfnkurs.gfn.internal
  DocumentRoot /var/www/gfnkurs/
  ErrorLog /var/log/apache2/gfnkurs-error.log
  CustomLog /var/log/apache2/gfnkurs-access.log combined
</VirtualHost>
EOF
```

## Enable site and reload Apache
```bash
sudo a2ensite gfnkurs.gfn.internal.conf
sudo systemctl reload apache2

```

## DNS Configuration on SRV1

```bash
# Add DNS record
sudo nano /etc/bind/db.gfn.internal

# Add:
gfnkurs    IN    A    192.168.10.13
```

## Test
http://gfnkurs.gfn.internal

