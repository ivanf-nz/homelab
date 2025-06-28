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

This homelab is built around a Topton N100 mini PC and manages networking, DNS filtering, virtualised services, and secure remote access. It gives me full control over the infrastructure in my flat, and a place to learn and experiment with networking and sysadmin workflows.

### pfSense

pfSense is an open-source firewall and router OS, running as a virtual machine on Proxmox. It handles:

- Routing across multiple subnets (LAN, WLAN, SWITCH_LAN, VPN)
- Static DHCP assignments and DNS forwarding
- VLAN separation and firewall policy enforcement
- WireGuard VPN for remote secure access to my local network

### Proxmox VE

Proxmox VE is installed directly on the Topton and manages:

- VMs (like pfSense) and LXC containers (like Pi-hole and Nginx)
- Snapshots, resource usage, and container lifecycle
- Web UI (default port `8006`) for complete system control

### Pi-hole

Pi-hole runs in a container and acts as the DNS resolver for all clients:

- Filters ads, trackers, and telemetry network-wide
- Provides local DNS for internal services (like `proxmox.local`)
- Is set as the primary DNS server through pfSense

### Nginx

Nginx is used as a reverse proxy for internal web services:

- Enables clean HTTPS access to services like Proxmox and Pi-hole without needing to remember ports or IPs
- Works with Pi-hole to resolve domains like `proxmox.local` and route them to the correct internal service and port
- Runs in an LXC container on Proxmox

### WireGuard

WireGuard is configured within pfSense to allow remote VPN access:

- My devices can connect back to the homelab from anywhere
- All DNS requests are tunnelled through Pi-hole
- Internal services remain accessible via their `.local` hostnames even off-site

---

## 🌐 Network Segmentation

The network is split into multiple VLAN-backed subnets, each with a dedicated purpose. pfSense manages routing and firewall rules between them:

| Subnet         | CIDR              | Purpose                                             |
|----------------|-------------------|-----------------------------------------------------|
| LAN            | `192.168.1.0/24`  | Primary subnet for all Proxmox-hosted services      |
| WLAN           | `192.168.2.0/24`  | Wi-Fi devices connected via TP-Link AX1500 AP       |
| SWITCH_LAN     | `192.168.3.0/24`  | Physical Ethernet devices on the Omada switch       |
| VPN            | `192.168.4.0/24`  | WireGuard tunnel network for remote device access   |

> **What is CIDR?**  
> CIDR stands for Classless Inter-Domain Routing. It is a way to specify IP address ranges more efficiently than the old classful system.  
> The `/24` means the first 24 bits of the IP address are the network part, so `192.168.1.0/24` includes all addresses from `192.168.1.0` to `192.168.1.255` (256 addresses total).  
> This is a common subnet size for local networks.

---

## 🧰 Hardware & Rack

All components are mounted in a compact 3D-printed 10-inch server rack using:

- [Modular 10‑inch server rack (reworked)](https://www.printables.com/model/1090551-modular-10-inch-server-rack-reworked)  
- [Modular 10‑inch server rack (220 × 200 bed variant)](https://www.printables.com/model/1173071-modular-10-inch-server-rack-reworked-220-x-200-bed)  

Rack contents:
- Topton N100 mini PC (bare-metal Proxmox host)
- 1U gigabit TP-Link Omada 5-port managed switch
- Raspberry Pi for testing and future utility use
- Power distribution
- JetKVM (mounted) for BIOS-level remote access

---

## 🧪 JetKVM for Headless Setup

To set up the Topton without plugging in a monitor, keyboard, or mouse, I used a **JetKVM** — a USB-powered device that emulates HDMI, keyboard, and mouse over the network. I used it to:

- Access the BIOS and change boot settings remotely
- Install Proxmox completely headless
- Get back into the system in case of failure or config lockout

The JetKVM is mounted in the rack permanently for easy remote recovery.

![Proxmox setup with JetKVM](https://github.com/user-attachments/assets/e9649b8f-1223-4ae8-b0f6-94099b4ad11a)

---

## 📡 Network Upgrades

Originally I tried to use a basic off-the-shelf router, but the wireless coverage was poor — lots of interference and dropout in my flat. DNS queries were laggy and devices would regularly fall off the network.

I upgraded to a **TP-Link AX1500 Wi-Fi 6 router** configured as an access point only (routing is done by pfSense). That improved stability and performance massively.

I also added a **TP-Link Omada 5-port managed switch**, which gives me VLAN support and better physical link visibility for debugging and network planning.

---

## 🖧 DNS and Local Routing

Internal services like Proxmox run on ports like `8006`. Rather than typing IPs and ports manually, I set up Nginx to reverse proxy services behind clean domains.

- Pi-hole handles internal DNS entries like:
  - `proxmox.local` → Nginx container IP
  - `pihole.local`  → Nginx container IP
- Nginx forwards the request to the correct internal port and IP

This lets me access services securely with simple, memorable URLs over HTTPS.

---

## 🔄 Future Plans

- Add Home Assistant for local automation, presence detection, and energy monitoring
- Eventually mount a small screen to the front of the rack to display system metrics or service status
