# Keycloak

Shared identity provider configuration for the project ecosystem, hosted at
`https://auth.bluebraces.online`.

Shared service documentation: [Recron project docs](https://docs.bluebraces.online/projects/keycloak/).

## Realm exports

Recron and QuantCore use separate realms on the same Keycloak instance:

| System | Realm | Configuration export |
| --- | --- | --- |
| Recron | `recron` | `realms/recron-realm-export.json` |
| QuantCore | `quantcore` | `realms/quantcore-realm-export.json` |

Both realms were configured manually in the Keycloak Admin Console. Their JSON
exports record those settings and follow the same deployment process. To record
a configuration change, update the appropriate realm in the Admin Console,
export it, and replace its corresponding file in this repository.

## Shared deployment

The **Deploy Keycloak** workflow runs on pushes to `main` that change the workflow
itself, `docker-compose.yml`, `realms/*.json`, or `themes/**`. It copies the Compose
file, both realm exports, and themes to `/opt/keycloak` on the VPS, then deploys
the shared Keycloak container and checks its health and HTTPS endpoint.

The workflow uses the `production` environment with these secrets:
`VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`, `KEYCLOAK_ADMIN`, and
`KEYCLOAK_ADMIN_PASSWORD`.

Compose mounts the shared `realms/` directory into the container. At startup,
the container copies the JSON files into Keycloak's import directory and starts
Keycloak with `--import-realm`. Existing realms are skipped by startup import,
so deploying an updated export does not apply its changes to an existing realm.
Manage changes to the running realms through the Admin Console.
See [Keycloak realm import](https://www.keycloak.org/server/importExport).
