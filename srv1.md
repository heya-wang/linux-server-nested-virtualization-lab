# SRV1 – Base System Setup

This section describes the **initial deployment and base configuration of SRV1**.

---

## 1. Virtual Machine Configuration

Create a VM in Proxmox with the following parameters.

| Parameter | Value                  |
| --------- | ---------------------- |
| Name      | SRV1                   |
| OS        | Debian                 |
| CPU       | 2 cores (type: `host`) |
| Memory    | 2048 MB                |
| Boot      | Start at boot enabled  |

The **Start at boot** option ensures SRV1 automatically starts when the Proxmox host reboots.

---

## 2. Network Configuration

During Debian installation, configure the network manually (no DHCP available yet).

| Parameter   | Value               |
| ----------- | ------------------- |
| IP Address  | 192.168.10.11       |
| Subnet Mask | 255.255.255.0       |
| Gateway     | 192.168.10.1        |
| DNS Server  | 1.1.1.1 (temporary) |
| Hostname    | srv1                |
| Domain      | gfn.internal        |

---

## 3. Software Selection

Install only the required server components:

- standard system utilities
- SSH server

No desktop environment should be installed.

---

## 4. Post-Installation Setup

Log in as `root` and grant administrator privileges to the `student` user.

```bash
apt update && apt install sudo -y
usermod -aG sudo student
