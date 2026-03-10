# Keycloak

Shared Keycloak identity provider for all BluesBraces projects (Recron, QuantCore, etc.).

Runs as a standalone Docker Compose stack on the VPS at `/opt/keycloak`, exposed via Traefik at `https://auth.bluebraces.online`.

## Structure

```
├── docker-compose.yml    # Keycloak service + Traefik labels
├── .env.example          # Template for secrets
├── realms/               # Realm exports (auto-imported on first start)
│   └── realm-export.json
└── themes/               # Custom login themes (keywind)
    └── keywind/
```

## Deployment

```bash
# On VPS
cd /opt/keycloak
cp .env.example .env     # Edit with real credentials
docker compose up -d
```

Traefik auto-discovers the container via Docker labels and provisions TLS certificates.

## Realms

| Realm | Used by |
|-------|---------|
| `recron` | Recron (app.bluebraces.online) |

New projects should create their own realm or reuse `recron` for SSO.
