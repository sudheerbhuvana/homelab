# Devices

A small homelab built with the gadgets I already had and whatever I could afford.

## Compute & Cloud

| Device | OS | CPU | RAM | Storage | Network | Role |
|---|---|---|---|---|---|---|
| **Local Nodes** | | | | | | |
| [Ryzen Node (PVE)](#ryzen-node-pve) | Proxmox VE | Ryzen 3 3200G (Vega 8) | 24 GB DDR4 3200MHz (`1x16 + 1x8`) | 128 GB SSD + 1 TB SanDisk Portable SSD | 2x 1GbE | Hypervisor (LXC containers & VMs) |
| [HP T530](https://www.hp.com/us-en/shop/pdp/hp-t530-thin-client-p-7fq52ut-aba-1) (ZIMA-OS) | ZimaOS | AMD GX-215JJ (2C, 1.5–2.0 GHz) | 4 GB DDR4 | 240 GB SSD + 1 TB HDD | 1GbE | NAS, small apps & media (Servarr stack) |
| [Dell Wyse 3040](https://www.dell.com/en-in/shop/wyse-endpoints-and-software/wyse-3040-thin-client/spd/wyse-3040-thin-client) (WYSE-P) | Debian Minimal | Intel Atom x5-Z8350 (4C, 1.44–1.92 GHz) | 2 GB DDR3L | 8 GB eMMC | 1GbE | Network-wide reverse proxy (NPM), primary DNS (Pi-hole 1), Cloudflared |
| [Dell Wyse 3040](https://www.dell.com/en-in/shop/wyse-endpoints-and-software/wyse-3040-thin-client/spd/wyse-3040-thin-client) (WYSE-S) | Debian Minimal | Intel Atom x5-Z8350 (4C, 1.44–1.92 GHz) | 2 GB DDR3L | 8 GB eMMC | 1GbE | Secondary DNS (Pi-hole 2), Cloudflared (main use TBD) |
| **Cloud (Oracle Free Tier)** | | | | | | |
| [Oracle Cloud](https://www.oracle.com/in/cloud/free/) (CLOUD-NANO) | Ubuntu 24.04 Minimal | 1/8 OCPU (AMD EPYC 7551) | 1 GB | 50 GB | 50 Mbps | Always-on monitoring & management (Glance, Arcane, Pi-hole 3) |
| [Oracle Cloud](https://www.oracle.com/in/cloud/free/) (oci-amd-1) | Ubuntu 24.04 Minimal | 1/8 OCPU (AMD EPYC 7551) | 1 GB | 100 GB | 50 Mbps | WireGuard tunnel (`wg-easy`) for Servarr / torrent traffic |
| [Oracle Cloud](https://www.oracle.com/in/cloud/free/) (oci-amd-2) | Ubuntu 24.04 Minimal | 1/8 OCPU (AMD EPYC 7551) | 1 GB | 100 GB | 50 Mbps | Cron jobs & temp backups (planned) |

## Networking, Power & Gadgets

| Device | Type | Specs | Role |
|---|---|---|---|
| **Networking** | | | |
| [TP-Link ES208G](https://amzn.in/d/0aRFohPV) | Managed Switch | 8x Gigabit ports, fanless metal, Omada cloud managed | Main switch |
| [TP-Link TL-SG105E](https://amzn.in/d/0dxaZiNP) | Smart Switch | 5x Gigabit ports, VLAN, QoS, IGMP, link aggregation | Secondary switch for extra devices |
| [TP-Link Archer C6 (AC1200)](https://amzn.in/d/03ZuGXe5) | Wi-Fi Router / AP | 867 Mbps (5 GHz) + 400 Mbps (2.4 GHz), 5x Gigabit ports | Main router & AP (WAN from ISP router) |
| **Power** | | | |
| [APC Back-UPS BX600C-IN](https://amzn.in/d/0hx4ceLl) | UPS | 600VA / 360W, 3 outlets | ~20 min backup for main switch, PVE & WYSE-P |
| Router UPS *(model TBD)* (x2) | Mini UPS | Separate battery backups | Keeps ISP router & Archer C6 online |
| [Tapo P110](https://amzn.in/d/0hs3AECn) | Wi-Fi Smart Plug | 16A, energy monitoring | Power monitoring & control |
| **Monitor, Keyboard & Cameras** | | | |
| [Acer EK240Y P6](https://amzn.in/d/0iCqTYeg) | Monitor | 23.8" 1080p IPS, 100Hz, HDMI + VGA | Display for local access |
| [HP KM 180](https://amzn.in/d/0gqhDY6f) | Keyboard & Mouse | Wired USB, full-size, 1200 DPI | Input for local access |
| [TP-Link Tapo C200](https://www.tp-link.com/in/home-networking/cloud-camera/tapo-c200/) (x2) | Wi-Fi Camera | 1080p, 360° pan / 114° tilt, IR night vision, RTSP/ONVIF | Home surveillance |

The entire setup is connected to the home inverter. Even if the inverter goes down, the **APC Back-UPS** gives around 20 minutes of backup for the **main switch**, **PVE**, and **WYSE-P**, while the **ISP router** and **Archer C6** stay online on their own router UPS units.

---

## Ryzen Node (PVE)

<!-- ![Ryzen Node (PVE)](images/pve.jpg) -->

This was my very first PC, now repurposed as a Proxmox server for LXC containers and virtual machines. When the old motherboard stopped working, I replaced it with an ASRock B450M board, added another 16 GB RAM stick, and put in extra cooling fans:

- **Motherboard:** [ASRock B450M-HDV R4.0](https://amzn.in/d/0hXTsmtx) (AM4, mATX)
- **CPU & Cooler:** [AMD Ryzen 3 3200G](https://amzn.in/d/0i16vl6U) (Vega 8) + Ant Esports ICE-C612 V2 ARGB
- **RAM:** [24 GB Corsair Vengeance LPX DDR4 3200MHz CL16](https://amzn.in/d/07SOLLxF) (`1x16GB` + `1x8GB`)
- **Storage:** 128 GB SSD + [SanDisk 1 TB Portable SSD](https://amzn.in/d/0fG1iE3L) (USB 3.2 Gen 2)
- **NIC:** [H!Fiber Intel 82576 Dual-Port 1G Gigabit NIC](https://amzn.in/d/0fAuUKzX)
- **PSU:** [Cooler Master MWE 450 Bronze V2 230V](https://www.coolermaster.com/en-in/products/mwe-450-bronze-v2-230v.html) (450W, 80+ Bronze)
- **Case & Fans:** [Ant Esports Elite 1000 PS](https://amzn.in/d/0ct8ADTc) + [5x Ant Esports Darkflow 120mm PWM fans](https://amzn.in/d/0ftxFsfO)

---

<details>
<summary>Cables & Power Strips</summary>

- **Power strips:** 2x [GM 3060 extension boards](https://amzn.in/d/01WkJvex) (`10A`, `250V`, 4 sockets, 2m cord)
- **Display switching:** [Sounce 2-port passive HDMI switch](https://amzn.in/d/06qqLRi3) (4K 60Hz): 1x HDMI from switch to the Acer monitor, 1x HDMI from PVE to the switch, and 1x DisplayPort-to-HDMI cable shared across the HP T530 and Wyse nodes
- **Ethernet cables:** 4x [Cat6 patch cables](https://www.amazon.in/dp/B0FXB5CJJQ) (`0.5 m`), 8x [FEDUS Cat5e cables](https://amzn.in/d/06ePLcYr) (`1 m`), 1x [UGREEN Cat 6 braided](https://amzn.in/d/0aNwDgoq) (`2 m`, Proxmox dual LAN), and 1x [UGREEN Cat 7 flat](https://amzn.in/d/0hg8u1K2) (`3 m`, run to the TV)

</details>
