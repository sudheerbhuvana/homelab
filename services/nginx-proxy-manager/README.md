# Nginx Proxy Manager

Network-wide reverse proxy and SSL certificate manager running on Docker (`jc21/nginx-proxy-manager:latest`) on `wyse-primary` (Dell Wyse 3040).

## Ports

- `80`: HTTP traffic
- `81`: Admin Web UI
- `443`: HTTPS traffic

## Quick Start

```bash
docker compose up -d
```

- Initial Admin URL: `http://<ip>:81`
- Persistent SQLite database and configuration are stored in `./data`.
- Let's Encrypt certificates and private keys are stored in `./letsencrypt`.
