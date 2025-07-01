# 🏡 Homelab

A self-hosted homelab running on local hardware using virtual machines, containers, and open-source tools.

## 🛠️ Services

All services are hosted on a **Topton N100 mini PC** running **Proxmox VE**. Proxmox manages the virtual machines and LXC containers that make up the homelab stack:

| Service     | Proxmox Type        | Purpose                                           |
|-------------|---------------------|---------------------------------------------------|
| pfSense     | Virtual Machine (VM)| Router, firewall, DHCP, WireGuard VPN             |
| Pi-hole     | LXC Container       | DNS resolver and network-wide ad blocking         |
| Nginx       | LXC Container       | Reverse proxy for internal services with HTTPS    |
| Proxmox VE  | Bare Metal          | Hypervisor for managing all VMs and LXCs          |

---

## ⚙️ Overview

This homelab is built around a **Topton N100 mini PC**, running [Proxmox VE](https://www.proxmox.com) directly on bare metal. All core services—firewalling, DNS filtering, VPN, and local apps—are virtualised inside Proxmox using either VMs or LXC containers.

It gives me complete control over networking in my flat, offers isolation between services, and serves as a hands-on platform to learn sysadmin, networking, and Linux infrastructure.

---

### pfSense (VM)

[pfSense](https://www.pfsense.org/) is an open-source firewall and router OS, running as a **virtual machine** inside Proxmox. It handles:

- Routing across all subnets (LAN, WLAN, SWITCH_LAN, VPN)
- DNS forwarding and static DHCP assignments
- VLAN separation and firewall rules
- WireGuard VPN endpoint for secure remote access

---

### Proxmox VE (Host)

[Proxmox VE](https://www.proxmox.com) is installed directly on the Topton and provides:

- A full virtualisation stack with a browser-based UI
- Management of VMs (like pfSense) and LXC containers (like Pi-hole, Nginx)
- Snapshots, backups, resource usage, and live monitoring
- Access to internal services via port `8006` (e.g. `https://proxmox.local:8006`)

---

### Pi-hole (LXC)

[Pi-hole](https://pi-hole.net/) runs as an LXC container and acts as the main DNS server for the network:

- Filters ads, trackers, and telemetry across all devices
- Resolves local domains like `proxmox.local` and `pihole.local` to Nginx
- All client DNS is routed through Pi-hole via pfSense to ensure consistent filtering and local domain resolution

---

### Nginx (LXC)

[Nginx](https://nginx.org/) is used as a reverse proxy, also running as a container:

- Routes friendly domain names (like `proxmox.local`) to their proper internal services
- Exposes internal services (like :8006) through clean HTTPS URLs, removing the need to remember ports
- DNS for each local domain is resolved by Pi-hole, then routed internally via Nginx

---

### WireGuard (in pfSense)

[WireGuard](https://www.wireguard.com/) is configured inside pfSense to provide encrypted remote access:

- My devices can tunnel back into the homelab from any network
- All DNS is routed through Pi-hole via the tunnel
- Local services are still accessible using `.local` hostnames

---

## 🌐 Network Segmentation

The network is split into multiple VLAN-backed subnets, each with a specific role. pfSense manages routing between them and enforces rules via firewall policies.

| Subnet         | CIDR              | Purpose                                             |
|----------------|-------------------|-----------------------------------------------------|
| LAN            | `192.168.1.0/24`  | Proxmox-hosted services (Pi-hole, Nginx, etc.)      |
| WLAN           | `192.168.2.0/24`  | Wi-Fi clients via TP-Link AX1500 AP                 |
| SWITCH_LAN     | `192.168.3.0/24`  | Wired clients connected to the Omada switch         |
| VPN            | `192.168.4.0/24`  | WireGuard VPN clients from external networks        |

> **What is CIDR?**  
> CIDR (Classless Inter-Domain Routing) defines IP address ranges.  
> A subnet like `192.168.1.0/24` uses the first 24 bits for the network portion, covering `192.168.1.0` to `192.168.1.255`.  

---

## 🧰 Hardware & Rack

Everything is mounted inside a compact 3D-printed 10-inch rack:

- [Modular 10‑inch server rack (reworked)](https://www.printables.com/model/1090551-modular-10-inch-server-rack-reworked)  
- [Modular 10‑inch server rack (220×200 bed variant)](https://www.printables.com/model/1173071-modular-10-inch-server-rack-reworked-220-x-200-bed)  

### Rack contents:

- **Topton N100 mini PC** (bare-metal Proxmox host)
- **TP-Link Omada 5-port managed switch** (VLAN support)
- **Raspberry Pi** (for testing and future projects)
- **Power distribution** (USB-C and 12V rail)
- **JetKVM** (for BIOS-level headless access)

---

## 🧪 JetKVM for Headless Setup

To set up the Topton without plugging in a display, keyboard, or mouse, I used a **JetKVM** — a network-based remote KVM-over-IP device that emulates HDMI, keyboard, and mouse.

I used it to:

- Access the BIOS remotely
- Perform the full Proxmox installation with no physical monitor
- Recover from potential lockouts or system config issues

The JetKVM is now permanently mounted in the rack for easy access and recovery.

![Proxmox setup with JetKVM](https://github.com/user-attachments/assets/e9649b8f-1223-4ae8-b0f6-94099b4ad11a)

---

## 📡 Network Upgrades

Originally I started with a basic consumer router, but the Wi-Fi coverage was poor. Interference in my flat caused devices to drop off or DNS queries to stall.

Upgrades:

- Switched to a **TP-Link AX1500 Wi-Fi 6 router**, now running in **access point mode** (routing is done by pfSense)
- Added a **TP-Link Omada 5-port managed switch**, giving me:
  - VLAN support
  - Port tagging and trunking
  - Easier debugging with per-port stats

---

## 🖧 DNS and Local Routing

Many internal services (like Proxmox) use "non-standard" ports, e.g. `:8006`. To simplify access:

- I configured **Pi-hole** to resolve domains like:
  - `proxmox.local` → Nginx container IP
  - `pihole.local`  → Nginx container IP
- Nginx listens on port 443 and proxies each domain to the appropriate container and port
- This allows access to any internal service via clean HTTPS domains, even remotely through the VPN

---

## 🔄 Future Plans

- Add **Home Assistant** for smart home automation, presence detection, and dashboards
- Eventually mount a small **status screen** on the front of the rack (e.g. Pi-driven) to display temperatures, network stats, and service uptime
