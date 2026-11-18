# Services

Each service in the homelab is deployed as a self-contained folder under
`services/` containing its compose file, config and assets.

| Service | Purpose | Notes |
|---|---|---|
| [glance](glance/) | Dashboard | Aggregates status from every other service |
| [pocket-id](pocket-id/) | Identity Provider (OIDC) | Lightweight OIDC provider for homelab authentication |
| [tinyauth](tinyauth/) | Forward Auth | Lightweight authentication proxy with Pocket ID OAuth support |
