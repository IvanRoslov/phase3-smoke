# Lessly Phase 3 smoke test

Minimal compose stack that exercises the full Phase 3 (compose-first
multi-service) flow on a Lessly tenant namespace.

## What it tests

- **Non-root images** — `bitnami/redis`, `bitnami/postgresql`, and
  `nginxinc/nginx-unprivileged` all run as non-root UIDs, so the
  PSA-restricted namespace lets them start.
- **`${VAR}` compose variables** — `DB_PASSWORD` (secret) and `TAG`
  surface in the import wizard's vars step.
- **Named volume** — `db_data` is mounted into `db` as `/bitnami/postgresql`,
  provisioning a PVC and showing as an inline mount strip on the `db`
  service card.
- **`depends_on` chain** — `api` waits for `cache:6379` and `db:5432` via
  the bundled `wait-for-port` initContainer before its main container
  starts.
- **Private DNS** — `api` resolves `db` and `cache` by their slugs inside
  the tenant namespace.

## Apply

In the Lessly canvas, click **Add service → docker-compose.yml** and fill in:

| Field    | Value                                          |
|----------|------------------------------------------------|
| Repo URL | https://github.com/IvanRoslov/phase3-smoke     |
| Branch   | main                                           |
| Path     | compose.yaml                                   |

Then in the vars step:

| Var          | Value         | Secret |
|--------------|---------------|--------|
| DB_PASSWORD  | (any string)  | ✅     |
| TAG          | v1            | —      |

Click **Apply & deploy**. Three services + one volume should land in the
namespace, with `api` blocking on its initContainer until `cache` and `db`
are up.
