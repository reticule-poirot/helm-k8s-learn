# helm-k8s-learn

A Helm chart for deploying the [`go-docker-learn`](https://github.com/reticule-poirot/go-docker-learn) app
to Kubernetes. Built while learning Helm and Kubernetes basics — started
from `helm create` scaffolding, since customized for this app.

## Installing

```bash
helm install my-release . \
  --set image.repository=myapp \
  --set image.tag=0.0.1
```

If `image.tag` is left unset, it defaults to `appVersion` in `Chart.yaml`
(currently `0.0.1`) — keep that in sync with whatever tag you built the
image as (e.g. `docker build -t myapp:0.0.1 .` in the app repo's README).

For a local `kind` cluster, `myapp` doesn't need a registry prefix — load
the image directly with `kind load docker-image myapp:0.0.1 --name desktop`
(see the app repo's README) before installing. If you're using Docker
Desktop's built-in Kubernetes instead of `kind`, no load step is needed —
it shares the same Docker daemon, so the locally-built image is already
visible. For a real cluster, set `image.repository` to your
registry-qualified image name instead.

## Key configuration

| Value                          | Default | Notes                                                        |
|---------------------------------|---------|---------------------------------------------------------------|
| `service.port`                 | `80`    | External-facing Service port                                  |
| `service.targetPort`           | `8000`  | Container port — kept separate from `service.port` because the app runs as a non-root user and can't bind ports below 1024 |
| `autoscaling.minReplicas`      | `1`     |                                                                 |
| `autoscaling.maxReplicas`      | `5`     | Kept modest — this is a learning chart, not sized for production scale |
| `podSecurityContext` / `securityContext` | non-root, read-only root filesystem, all capabilities dropped | Matches the app's distroless `nonroot` base image |

`HOSTNAME` and `NODE_NAME` are injected into the container via the
Kubernetes Downward API (pod name and node name respectively) — Kubernetes
does not set these automatically. See `templates/deployment.yaml`.

## Probes

- **Liveness** (`/health`): process alive, no dependency checks.
- **Readiness** (`/ready`): reflects whether the app is actually serving
  traffic; goes false immediately on shutdown so in-flight requests drain
  before the pod stops receiving new ones.

Both carry explicit `initialDelaySeconds`/`periodSeconds`/`failureThreshold`
tuning in `values.yaml` as defense-in-depth, on top of a startup race that
was already fixed at the application level.

## Development

- `helm lint .` and `helm template .` before committing — see `CLAUDE.md`
  for the specific things most likely to silently break if refactored
  (port split, Downward API env vars, security context).
