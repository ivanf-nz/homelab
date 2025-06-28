# 🏡 Homelab

A self-hosted homelab running on local hardware using virtual machines, containers, and open-source tools.

## ⚙️ Overview

- **Firewall & Routing**: [pfSense](https://www.pfsense.org/) running on a Topton N100 mini PC.  
- **Virtualisation Host**: [Proxmox VE](https://www.proxmox.com/en/) manages VMs and LXC containers.  
- **DNS & Ad-blocking**: [Pi-hole](https://pi-hole.net/) filters DNS requests network-wide.  
- **Reverse Proxy**: [Nginx](https://nginx.org/) handles HTTPS with SSL certs and internal routing.  
- **VPN**: [WireGuard](https://www.wireguard.com/) allows remote access and ad‑blocked DNS for my devices on any network.  

## 🖧 Network Layout

Multiple subnets (LAN, WLAN, SWITCH_LAN) are set up with pfSense managing firewall rules and DHCP. DNS from all networks is routed through Pi-hole.

## 🧰 Hardware & Rack

Everything is mounted in a 3D‑printed 10‑inch server rack based on community designs:  
- [Modular 10‑inch server rack (reworked)](https://www.printables.com/model/1090551-modular-10-inch-server-rack-reworked)  
- [Modular 10‑inch server rack (220 × 200 bed variant)](https://www.printables.com/model/1173071-modular-10-inch-server-rack-reworked-220-x-200-bed)  

The rack holds the Topton N100, switch, Raspberry Pi, and other gear. A small screen is attached for local status display.

## 🔐 Remote Access

WireGuard is set up on pfSense to provide VPN access for all my devices, letting them use Pi-hole and access services securely from anywhere.

## 🛠️ Services

| Service     | Host       | Purpose                         |
|-------------|------------|---------------------------------|
| pfSense     | Topton N100| Router, Firewall, WireGuard VPN |
| Proxmox VE  | Mini PC    | VM and container management     |
| Pi-hole     | LXC        | Network-wide DNS and ad‑block   |
| Nginx       | LXC        | Reverse proxy with SSL          |

## 🔄 Future Plans

- Add Home Assistant  
- Add a screen on the rack to monitor the services  
