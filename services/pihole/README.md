# Pi-hole

Network-wide DNS sinkhole and ad-blocker running on Docker (`pihole/pihole:latest`), deployed in a redundant multi-instance setup across three nodes.

## Deployment Nodes

| Instance | Node | IP / Network | Web Admin URL | Role |
|---|---|---|---|---|
| **Pi-hole 1** | `wyse-primary` (Dell Wyse 3040) | `10.0.0.231` | `https://pihole1.sudheer.fyi` | Primary local LAN DNS |
| **Pi-hole 2** | `wyse-secondary` (Dell Wyse 3040) | `10.0.0.154` | `https://pihole2.sudheer.fyi` | Secondary local LAN DNS |
| **Pi-hole 3** | `cloud-nano` (Oracle Cloud) | `100.126.139.15` (Tailscale) | `https://pihole3.sudheer.fyi` | Remote / Tailnet DNS |

## Configuration Highlights

- **DNS Listening Mode (`ALL`):** Configured with `FTLCONF_dns_listeningMode: ALL` ("Permit all origins") so that queries from all network interfaces and VPN clients (like Tailscale) are accepted and resolved.
- **Upstream DNS:** `1.1.1.1` (Cloudflare) and `9.9.9.9` (Quad9).
- **Web Admin Interface:** Mapped to port `8080` (reverse proxied via Nginx Proxy Manager to `https://pihole1.sudheer.fyi`).

## Quick Start

1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
2. Set your web admin password (`PIHOLE_PASSWORD`) in `.env`.
3. Start the container:
   ```bash
   docker compose up -d
   ```
