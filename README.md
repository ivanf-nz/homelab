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

It gives me complete control over networking in my apartment, offers isolation between services, and serves as a hands-on platform to learn sysadmin, networking, Linux infrastructure and the Linux command line.

---

### pfSense (VM)

[pfSense](https://www.pfsense.org/) is an open-source firewall and router OS, running as a **virtual machine** inside Proxmox. It handles:

- Routing across all subnets (LAN, WLAN, SWITCH_LAN, WG_VPN)
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

- Routes friendly domain names (like `proxmox.local`) to their proper internal services + their associated port, as Pi-hole is unable to route to a ip + port
- Exposes internal services (like :8006) through clean HTTPS URLs, removing the need to remember ports (as mentioned above)
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
- **Raspberry Pi** (for future projects such as a screen monitoring the homelab or some other analysis tool)
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

Originally, I tried to set my ISP-provided modem to access point mode, but it didn’t support it. So I bought the cheapest standalone router I could find, but the Wi-Fi signal struggled with interference in my flat, and speeds dropped below 10 Mbps for both upload and download. I ended up upgrading to something stronger.

Upgrades:

- Switched to a **TP-Link AX1500 Wi-Fi 6 router**, now running in **access point mode** (routing is done by pfSense)
- Added a **TP-Link Omada 5-port managed switch**, giving me VLAN support in the future 

---

## 🖧 DNS and Local Routing

Many internal services (like Proxmox) use "non-standard" ports, e.g. `:8006`. To simplify access:

- I configured **Pi-hole** to resolve domains like:
  - `proxmox.local` → Nginx container IP  
  - `pihole.local`  → Nginx container IP  
- Nginx listens on port 443 and proxies each domain to the appropriate container and port  
- This allows access to any internal service via clean HTTPS domains, even remotely through the VPN  

---

## 🧩 3D Printed Panels

![📸 Photo placeholder – 3D homelab rack setup coming soon!](https://via.placeholder.com/800x450?text=Photo+of+3D-Printed+Homelab+Rack+Coming+Soon)

To make the setup neat and secure, I designed custom front panels for each rack-mounted device. These panels are 3D printed and designed to fit into the modular 10-inch rack system listed above.

### STL Files

All STLs are available in the **main directory of this repository**.

| Filename                        | Description                                                                 |
|--------------------------------|-----------------------------------------------------------------------------|
| `1u_panel_pi_kvm_v1.stl`       | 1U panel for a Raspberry Pi and JetKVM. The JetKVM fits tightly, and I manually punched a hole through the plastic, aligned with one of the Pi’s existing mounting holes, to screw it into place securely. |
| `1u_panel_topton_left.stl`     | Left side of the bottom section of the Topton N100 mini PC panel (split for easier 3D printing). |
| `1u_panel_topton_right.stl`    | Right side of the bottom section of the Topton N100 mini PC panel (split for easier 3D printing). |
| `1u_panel_topton_upper.stl`    | Top section of the Topton N100 mini PC panel (completes the full 2U front). |
| `1u_panel_ES205G.stl`          | 1U panel for the **TP-Link Omada 5-port managed switch (ES205G)**.         |

### Attribution

These models were built using base measurements and STLs provided by:

- [Raspberry Pi 10-inch Rack Mount by esluyter](https://www.printables.com/model/1185545-raspberry-pi-3b4b5b-10-inch-rack-mount) – used for Pi alignment and sizing  
- [Modular 10-inch Rack by MickMake](https://www.printables.com/model/1090551-modular-10-inch-server-rack-reworked/files) – used for the blank 1U rack faceplates and the overall rack frame  
- [RackStuds by Eirik Jeppesen](https://www.printables.com/model/1233475-rackstuds-for-m6-106-racks/files) – used for the 3D-printed M6 screws because proper server rack screws are very expensive  

---

## 🔄 Future Plans

- Add **Home Assistant** for smart home automation, presence detection, and dashboards  
- Eventually mount a small **status screen** on the front of the rack (e.g. Pi-driven) to display temperatures, network stats, and service uptime  
