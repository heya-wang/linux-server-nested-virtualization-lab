# Linux Server Infrastructure Lab with Nested Virtualization

## Overview

This project documents a virtualized Linux server environment built with nested virtualization.  
The lab simulates a small IT infrastructure where multiple Linux servers provide common network and application services.

The environment is designed to practice system administration, networking, and service deployment in a realistic infrastructure scenario.

Key technologies include:

- Linux server administration
- Network services (DNS, DHCP, LDAP)
- Web and collaboration services
- Containerized applications
- Monitoring and backup
- Nested virtualization using Proxmox

---

## Architecture Overview

The lab environment runs on a physical host using Hyper-V.  
Inside the virtual environment, Proxmox VE provides nested virtualization to host multiple Linux servers and infrastructure services.

![Architecture Overview](diagrams/topology.png)

---

## Network Design

The environment contains two main networks:

- **LAN Network** – internal communication between servers
- **Management Network** – administrative access via jump host (CL01)

The management network allows SSH access to all servers through the jump host.

---

## Server Roles

| Server | Services |
|------|------|
| **Srv1** | DNS, DHCP, LDAP |
| **Srv2** | Web Server, Nextcloud, Backup, Docker Containers，FTP |
| **Srv3** | Mail Server |

---

## Infrastructure Components

### Router
Provides routing between the internal network and the external network.

### CL01 – Jump Host
Used for administrative access to internal servers through the management network.

### Zabbix
Monitoring system used to observe system health and service availability.

### Storage
Provides storage resources for services and backups.

### Proxmox VE
Used as a nested virtualization platform to host additional Linux servers.

---

## Implemented Services

**Network Services**

- DNS
- DHCP
- LDAP

**Application Services**

- Web Server
- Nextcloud
- Mail Server

**Infrastructure Services**

- Backup system
- Docker container environment
- Monitoring with Zabbix

---

## Purpose of the Project

This lab environment was created to:

- practice Linux system administration
- deploy and configure typical enterprise services
- understand network architecture and service dependencies
- experiment with virtualization and containerization
- simulate a small enterprise IT infrastructure

---

## Technologies Used

- Linux
- Proxmox VE
- Hyper-V
- Docker
- Zabbix
- Nextcloud
- DNS / DHCP
- LDAP
- Mail Server

---

## Repository Structure
