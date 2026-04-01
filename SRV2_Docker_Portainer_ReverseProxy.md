# Docker Installation & Container Management Training
## Docker Installation
Add Docker's official GPG key
```bash
sudo apt update
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker apt repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

Install Docker
```bash
# Docker installation
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Verify installation
sudo docker run hello-world

```

Add current user to docker group
```bash
sudo usermod -aG docker <benutzername>

```
---

## Basic Docker Commands
Image Management
```bash
# Search for images on Docker Hub
docker search debian

# Download an image
docker pull debian

# List local images
docker images
```
---

## Docker Container Management
```bash
# Start a container in interactive mode
docker run -it --name container1 debian

# Exit container (container stops)
exit

# Start a stopped container
docker start container1

# Re-enter a running container
docker attach container1

# Exit without stopping container
# Press: Ctrl+P Ctrl+Q

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop container1

# Delete a container
docker rm container1

# Delete all stopped containers
docker rm $(docker ps -a -q)

```
---

## Custom Web Server Container
Start container and install packages
```bash
# Start Debian container
docker run -it --name ref debian

# Inside container, install packages
apt update
apt install apache2 mc nano -y

# Exit container (container stops)
exit
```

Create new image from container
```bash
docker commit ref webref
```

Create customer directories and web pages
```bash
# Create customer directories
sudo mkdir /kunde1 /kunde2
```

Create web page for kunde1
```bash
sudo tee /kunde1/index.html > /dev/null << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Kunde 1</title></head>
<body><h1>Willkommen bei Kunde 1</h1></body>
</html>
EOF
```

Create web page for kunde2
```bash
sudo tee /kunde2/index.html > /dev/null << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Kunde 2</title></head>
<body><h1>Willkommen bei Kunde 2</h1></body>
</html>
EOF
```

Start container with Apache
```bash
docker run -itd --name kunde1 -p 8080:80 -v /kunde1:/var/www/html webref
docker exec -it kunde1 /etc/init.d/apache2 start

docker run -itd --name kunde2 -p 8081:80 -v /kunde2:/var/www/html webref
docker exec -it kunde2 /etc/init.d/apache2 start

# Verify Apache is running
docker exec kunde1 ps aux | grep apache
docker exec kunde2 ps aux | grep apache
```

Access the websites
Exit containers without stopping them
Press: Ctrl+P then Ctrl+Q

Access the websites from browser on CL01
http://192.168.10.13:8080  - displays kunde1's webpage
http://192.168.10.13:8081  - displays kunde2's webpage

---

## Portainer Installation
Create project directory and navigate into it
```bash
mkdir -p ~/docker/portainer
cd ~/docker/portainer
```

Create docker-compose.yml with Portainer configuration
```bash
cat > docker-compose.yml << 'EOF'
services:
    portainer:
      image: portainer/portainer-ce:latest
      container_name: portainer
      restart: unless-stopped
      security_opt:
        - no-new-privileges:true
      volumes:
        - /etc/localtime:/etc/localtime:ro
        - /var/run/docker.sock:/var/run/docker.sock:ro
        - ./portainer-data:/data
      ports:
        - 9000:9000
        - 9443:9443
EOF
```

Start and verify Portainer container
```bash
docker compose up -d
docker ps | grep portainer
```
 
Verify the result in the browser at https://192.168.10.13:9443
---

## Reverse Proxy & Port Forwarding
### DNS Configuration on SRV1
Add DNS entry on SRV1
```bash
# Connect to SRV1 and edit the BIND zone file
sudo nano /etc/bind/db.gfn.internal

# Add the following line:
portainer   IN   A    192.168.10.13

# reload BIND:
sudo systemctl reload bind9
```

### Apache Reverse Proxy Configuration
```bash
sudo tee /etc/apache2/sites-available/portainer.gfn.internal.conf > /dev/null << 'EOF'
<VirtualHost *:443>
    ServerName  portainer.gfn.internal
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
    SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
    ProxyPreserveHost On
    ProxyPass / https://localhost:9443/
    ProxyPassReverse / https://localhost:9443/
</VirtualHost>
EOF

# Enable the virtual host
sudo a2ensite portainer.gfn.internal.conf

# Enable Apache proxy modules
sudo a2enmod proxy proxy_http

# Enable Https connection
sudo a2enmod ssl proxy_connect

# Restart Apache
sudo systemctl restart apache2

# Verify the configuration
sudo apachectl configtest
```

Test the result
Access Portainer at: http://portainer.gfn.internal

### Router Port Forwarding on Router
```bash
# On the router, add iptables DNAT rule
sudo iptables -t nat -A PREROUTING -p tcp -d 10.100.17.32 --dport 443 -j DNAT --to-destination 192.168.10.13:9443

# Show NAT rules
sudo iptables -t nat -L PREROUTING -n -v

# Test external access
# On cl01
curl https://portainer.gfn.internal

```
---


