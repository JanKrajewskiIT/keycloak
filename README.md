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

### CI/CD (GitHub Actions)

Push to `main` triggers `.github/workflows/deploy.yml` which SSHs into the VPS, clones/pulls the repo via `GHCR_TOKEN`, and runs `docker compose up -d`.

Required secrets: `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`, `GHCR_TOKEN`, `KEYCLOAK_ADMIN`, `KEYCLOAK_ADMIN_PASSWORD`.

### Manual deployment

SSH into the VPS and run:

```bash
# Requires the "proxy" network (created by Traefik)
docker network inspect proxy >/dev/null 2>&1 || docker network create proxy

# Clone / update repo
cd /opt
git clone https://github.com/JanKrajewskiIT/keycloak.git keycloak 2>/dev/null || (cd keycloak && git pull)
cd /opt/keycloak

# Configure (first time only)
cp .env.example .env    # Edit KEYCLOAK_ADMIN and KEYCLOAK_ADMIN_PASSWORD

# Deploy
docker compose pull
docker compose up -d --remove-orphans
```

### Updating after changes

```bash
cd /opt/keycloak
git pull
docker compose pull
docker compose up -d --remove-orphans
```

## Realm import

The `realms/realm-export.json` is auto-imported on **first startup** only. If the realm already exists, the import is skipped (Keycloak built-in behavior).

To force re-import (e.g., after updating `realm-export.json`):

```bash
cd /opt/keycloak
docker compose down -v    # removes data volume — ALL realm data is lost
docker compose up -d      # fresh start with realm import
```

## Realms

| Realm | Used by |
|-------|---------|
| `recron` | Recron (app.bluebraces.online) |

New projects should create their own realm or reuse `recron` for SSO.
