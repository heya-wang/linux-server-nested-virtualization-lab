# Mail Server Installation on SRV3 (LXC Container)
## 1. LXC Container Setup

Create container on Proxmox with:

Template: debian-13 (Debian 13 Bookworm)

Hostname: srv3

Disk: 50 GiB

CPU: 2 cores

Memory: 1024 MiB

Swap: 1024 MiB

IPv4: 192.168.10.14/24

Gateway: 192.168.10.1

DNS Domain: gfn.internal

DNS Server: 192.168.10.11

---

## Basic Configuration

```bash
# Create user and install sudo
adduser student
apt update && apt install sudo -y
usermod -aG sudo student

# Install SSH and configure access
apt install openssh-server -y
```
---

##  Docker Installation
```bash

sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release -y

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt install docker-ce docker-ce-cli containerd.io -y
sudo apt install docker-compose-plugin -y

# Add user to docker group
sudo usermod -aG docker student

# Testing
docker --version
docker compose version

```
---

## Mail Server Setup
Create directory
```bash
mkdir -p ~/docker/mailserver
cd ~/docker/mailserver
```

Edit docker-compose.yml
```bash
cat > docker-compose.yml << 'EOF'
services:
  mailserver:
    image: ghcr.io/docker-mailserver/docker-mailserver:latest
    container_name: mailserver
    hostname: mail.gfn.internal
    env_file: mailserver.env
    ports:
      - "25:25"    # SMTP  (explicit TLS => STARTTLS)
      - "143:143"  # IMAP4 (explicit TLS => STARTTLS)
      - "465:465"  # ESMTP (implicit TLS)
      - "587:587"  # ESMTP (explicit TLS => STARTTLS)
      - "993:993"  # IMAP4 (implicit TLS)
    volumes:
      - ./docker-data/dms/mail-data/:/var/mail/
      - ./docker-data/dms/mail-state/:/var/mail-state/
      - ./docker-data/dms/mail-logs/:/var/log/mail/
      - ./docker-data/dms/config/:/tmp/docker-mailserver/
      - /etc/localtime:/etc/localtime:ro
      - ./cert/:/etc/owncert/
    restart: always
    stop_grace_period: 1m
    cap_add:
      - NET_ADMIN
    dns:
      - 192.168.10.11
      - 1.1.1.1
    healthcheck:
      test: "ss --listening --tcp | grep -P 'LISTEN.+:smtp' || exit 1"
      timeout: 3s
      retries: 0
EOF
```

Download and configure mailserver.env
```bash
wget https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master/mailserver.env

sudo tee mailserver.env > /dev/null << 'EOF'
DOMAIN=gfn.internal
HOSTNAME=mail.gfn.internal
POSTMASTER_ADDRESS=admin@gfn.internal
TZ=Europe/Berlin
SPOOF_PROTECTION=1
ENABLE_OPENDKIM=0
ENABLE_OPENDMARC=0
ENABLE_POLICYD_SPF=0
ENABLE_CLAMAV=1
ENABLE_RSPAMD=1
RSPAMD_LEARN=1
RSPAMD_GREYLISTING=1
RSPAMD_HFILTER=0
ENABLE_AMAVIS=0
SSL_TYPE=manual
SSL_CERT_PATH=/etc/owncert/cert.pem
SSL_KEY_PATH=/etc/owncert/key.pem
EOF
```

SSL Certificate
```bash
mkdir -p cert && cd cert
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 3650 -nodes -subj "/C=DE/ST=BaWue/L=Heidelberg/O=GFN/CN=mail.gfn.internal"
cd ..
chmod 644 cert/*.pem

```
---

## DNS Configuration  on srv1
```bash
# Add mail server records
cat >> /etc/bind/db.gfn.internal << 'EOF'

; Mail server
mail    IN  A     192.168.10.14
        IN  MX    10 mail.gfn.internal.
EOF

# Reload DNS service
sudo systemctl reload bind9

```
---

## Start Mail Server
```bash
# Check the status of postfix
sudo systemctl status postfix

# Stop local postfix
sudo systemctl stop postfix
sudo systemctl disable postfix

# Start mail server
cd ~/docker/mailserver
docker compose up -d

# Create email accounts
docker exec -it mailserver setup email add admin@gfn.internal <password>
docker exec -it mailserver setup email add student@gfn.internal <password>

# Check accounts
docker exec -it mailserver setup email list
```
---

## Mail Client Setup on cl01
Open Thunderbird on CL01, click "Weiter" – the wizard should automatically detect the correct settings.

Complete the wizard and confirm the trustworthiness of the self-signed certificate.

Set up the admin account (admin@gfn.internal) and student account (student@gfn.internal).

Test by sending a mail from admin to student and verifying receipt.

Optional: Research web-based email clients.

