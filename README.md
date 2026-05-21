# Lessly Phase 3 smoke test

Minimal compose stack that exercises the full Phase 3 (compose-first
multi-service) flow end-to-end.

## Apply

In the Lessly canvas, click **Add service → docker-compose.yml** and fill in:

| Field    | Value                                       |
|----------|---------------------------------------------|
| Repo URL | https://github.com/IvanRoslov/phase3-smoke  |
| Branch   | main                                        |
| Path     | compose.yaml                                |

Wizard prompts for one compose variable:

| Var  | Value | Secret |
|------|-------|--------|
| TAG  | v1    | —      |

Click **Apply & deploy**.

## What it tests

- **Non-root pods** — all three services run `nginxinc/nginx-unprivileged`
  (UID 101). The PSA-restricted namespace lets them start.
- **`${VAR}` compose variable** — `TAG` is captured in the wizard and
  substituted into `api`'s `VERSION` env at apply time.
- **Named volume** — `db_data` becomes a PVC mounted into `db` at
  `/tmp/data`; appears as an inline mount strip on the `db` canvas card.
- **`depends_on` ordering** — `api` blocks in its `wait-for-deps`
  initContainer until `cache:8080` AND `db:8080` accept TCP, then its
  main container starts.
- **Private DNS by slug** — wait-for-port resolves `cache` and `db` to
  their ClusterIP services without `service-<svcId>` prefixes.

All three are stand-ins — they're not real redis/postgres. The goal is to
verify the Phase 3 runtime mechanics (init ordering, PVC mount, intra-
namespace DNS) without dragging in non-root postgres/redis images, which
are surprisingly hard to find anonymously since Bitnami's August 2025
catalog change.

## When everything settles

```
kubectl -n lessly-<env-uuid> get pods
# Three Pods, all 1/1 Running.

kubectl -n lessly-<env-uuid> get pvc
# vol-<id[:8]>   Bound   1Gi  RWO  standard
```

Delete via the canvas: open the stack frame's side panel → **Source tab** →
**Delete stack**. Volume row remains in detached state; delete separately
via the volume node side panel.
