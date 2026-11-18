# Pocket ID

Lightweight OIDC provider for homelab authentication running on Docker (`pocketid/pocket-id:v2`).

## Quick Start

1. Copy `.env.example` to `.env` (if not already done):
   ```bash
   cp .env.example .env
   ```
2. Generate an encryption key:
   ```bash
   openssl rand -base64 32
   ```
3. Update `.env` with your domain (`APP_URL`) and generated `ENCRYPTION_KEY`.
4. Start the container:
   ```bash
   docker compose up -d
   ```
