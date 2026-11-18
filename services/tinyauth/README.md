# Tinyauth

Lightweight authentication and forward auth proxy running on Docker (`ghcr.io/tinyauthapp/tinyauth:v5`), integrated with Pocket ID for OIDC/OAuth authentication.

## Quick Start

1. Copy `.env.example` to `.env` (if not already done):
   ```bash
   cp .env.example .env
   ```
2. Configure Pocket ID OAuth client:
   - In Pocket ID, create a new OIDC client for Tinyauth.
   - Set the Redirect URI to:
     ```
     https://auth.sudheer.fyi/api/oauth/callback/pocketid
     ```
   - Copy the Client ID and Client Secret into `.env`:
     - `TINYAUTH_OAUTH_PROVIDERS_POCKETID_CLIENTID`
     - `TINYAUTH_OAUTH_PROVIDERS_POCKETID_CLIENTSECRET`
   - Update the Pocket ID URLs (`AUTHURL`, `TOKENURL`, `USERINFOURL`) to match your Pocket ID domain (e.g. `https://id.sudheer.fyi/...`).
3. Start the container:
   ```bash
   docker compose up -d
   ```

## Nginx Proxy Manager (NPM) Configuration

To protect any service in NPM using Tinyauth forward auth, paste the following into that proxy host's **Advanced** tab:

```nginx
location / {
    proxy_pass http://10.0.0.10:80;

    auth_request /tinyauth;
    auth_request_set $redirection_url $upstream_http_x_tinyauth_location;
    error_page 401 403 =302 $redirection_url;
}

location /tinyauth {
    internal;
    proxy_pass http://100.126.139.15:3000/api/auth/nginx;

    proxy_pass_request_body off;
    proxy_set_header Content-Length "";

    proxy_set_header X-Original-URL $scheme://$host$is_request_port$request_port$request_uri;
    proxy_set_header X-Original-Method $request_method;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
}
```

> [!NOTE]
> - Replace `http://10.0.0.10:80` with the actual backend IP and port of the protected service.
> - Ensure **Block Common Exploits** is turned **OFF** on the `auth.sudheer.fyi` proxy host in NPM so redirect query parameters are not stripped.

