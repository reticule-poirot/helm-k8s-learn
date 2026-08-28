# CLAUDE.md

Guidance for AI assistants (Claude Code or similar) working in this repo.

## What this is

A Helm chart deploying the Go app from the sibling repo `go-docker-learn`.
Standard `helm create` scaffold, since customized. This is a learning
project — keep defaults modest, not production-scale.

## Chart conventions

- Template helper prefix is `server.*` (`server.fullname`, `server.labels`,
  etc.), matching `Chart.yaml`'s `name: server`. Keep any new helpers under
  the same prefix — an earlier version had these accidentally named `..*`
  (a literal double-dot) from a broken chart-name substitution; don't
  reintroduce inconsistent naming.
- `values.yaml` comments explain *why*, not just *what* — preserve that
  when editing, especially around security context and port handling below.

## Known gotchas (already fixed once — don't reintroduce)

- **`service.port` vs `service.targetPort` are intentionally different**
  (80 vs 8000). The container runs as a non-root user (uid 65532, from the
  app's distroless `nonroot` image) and cannot bind privileged ports below
  1024. `deployment.yaml`'s `containerPort` and the app's `-port` flag must
  both reference `.Values.service.targetPort`, never `.Values.service.port`
  directly — collapsing these back into one value will crash-loop the pod.
- **`HOSTNAME` and `NODE_NAME` are wired via the Downward API** in
  `deployment.yaml`'s `env:` block (`metadata.name` / `spec.nodeName`).
  Kubernetes does not set either of these automatically as container env
  vars — if you see the app's `/` response show `node: unknown`, this is
  usually why. Don't remove these entries; the app's home handler depends
  on them.
- **Security context is deliberately non-empty.** `podSecurityContext` and
  `securityContext` in `values.yaml` set `runAsNonRoot`, fixed uid/gid
  65532, `readOnlyRootFilesystem: true`, all capabilities dropped, and
  `seccompProfile: RuntimeDefault` — these match the app's distroless image
  exactly. Don't revert to `{}` placeholders; if you add a container that
  needs different permissions, override per-container rather than loosening
  these defaults globally.
- **Readiness probe timing**: `readinessProbe`/`livenessProbe` in
  `values.yaml` carry explicit `initialDelaySeconds`/`periodSeconds`/
  `failureThreshold` as defense-in-depth. The actual startup race this
  guards against was fixed at the application level (see the Go repo's
  CLAUDE.md) — this tuning is a backstop, not a workaround for a live bug.

## Before committing

- `helm lint .` should pass cleanly.
- `helm template .` and spot-check that `service.targetPort`,
  `HOSTNAME`/`NODE_NAME` env vars, and the security context all render as
  expected — these three are the parts most likely to silently break if
  refactored.
