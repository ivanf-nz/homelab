# 🏡 Homelab

A self-hosted homelab running on local hardware using virtual machines, containers, and open-source tools.

## 🛠️ Services

| Service     | Host         | Purpose                                 |
|-------------|--------------|-----------------------------------------|
| pfSense     | Topton N100  | Router, firewall, DHCP, WireGuard VPN   |
| Proxmox VE  | Topton N100  | VM and container virtualisation         |
| Pi-hole     | LXC          | Network-wide DNS and ad‑blocking        |
| Nginx       | LXC          | Reverse proxy and local domain routing  |

---

## ⚙️ Overview

This homelab runs on a Topton N100 mini PC and is set up to manage internal networking, DNS filtering, virtualised services, and secure remote access. It provides full control over my infrastructure, both locally and remotely.

### pfSense

pfSense is an open-source router and firewall operating system that runs as a virtual machine on Proxmox. It handles all Layer 3 network functions:

- Assigns IP addresses via DHCP and manages multiple VLANs (LAN, WLAN, SWITCH_LAN)
- Enforces strict firewall rules between network segments
- Forwards DNS requests to Pi-hole for filtering
- Hosts the WireGuard VPN server for remote access
- Acts as the network gateway for all devices and services

pfSense provides fine-grained control over traffic, segmentation, and routing while keeping logs and metrics accessible through its web UI.

### Proxmox VE

Proxmox VE is the base operating system installed on the Topton N100. It manages all virtual machines and containers, including pfSense itself. It's responsible for:

- Running VMs (pfSense) and LXC containers (Pi-hole, Nginx, etc.)
- Handling backups, snapshots, and resource usage
- Offering a web UI to manage all services (on port `8006`)

### Pi-hole

Pi-hole is deployed as a lightweight container and acts as the DNS resolver for all devices. It:

- Blocks ads and trackers at the DNS level
- Helps reduce traffic to known malicious domains
- Resolves internal domain names like `proxmox.local`, routing them to the Nginx reverse proxy
- Works across all subnets by being integrated with pfSense as the default DNS server

### Nginx

Nginx runs in a container and acts as a reverse proxy for internal services. It allows clean, HTTPS-protected URLs like:

- `https://proxmox.local`
- `https://pihole.local`

These addresses route through Nginx to services running on non-standard ports (e.g. `8006` for Proxmox). To make this work, I added static DNS records in Pi-hole that point the domains to the IP of the Nginx container.

### WireGuard

WireGuard is a modern VPN protocol that is lightweight, fast, and easy to configure. It's set up on pfSense as a VPN server that:

- Allows my personal devices to securely connect to the homelab from anywhere
- Tunnels all DNS queries through Pi-hole even when on external networks
- Grants full access to internal services like Proxmox, Pi-hole, and Nginx-based apps as if I were on the LAN
- Uses static IP mapping and firewall rules to keep it secure and predictable

## 🖧 DNS and Local Routing

Many local services (like Proxmox) run on non-standard ports, which makes direct access awkward. To simplify things:

- I run Nginx as a reverse proxy and terminate SSL there
- I configure Pi-hole with DNS entries like:
  - `proxmox.local` → Nginx container IP
  - `pihole.local`  → Nginx container IP

Nginx then forwards requests to the actual services and ports behind the scenes. This makes local access clean, consistent, and secure.

## 🧰 Hardware & Rack

All hardware is mounted in a 3D-printed 10-inch server rack built from these open-source designs:
- [Modular 10‑inch server rack (reworked)](https://www.printables.com/model/1090551-modular-10-inch-server-rack-reworked)  
- [Modular 10‑inch server rack (220 × 200 bed variant)](https://www.printables.com/model/1173071-modular-10-inch-server-rack-reworked-220-x-200-bed)  

The rack contains:
- Topton N100 mini PC (running everything via Proxmox)
- A gigabit switch
- A Raspberry Pi for testing and monitoring
- Power management gear
- A small screen mounted on the front to show system stats

## 🔄 Future Plans

- Add Home Assistant for smart device control and automation
- Add screen for real-time service and network monitoring
