# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the Helm chart for deploying **Nutrient Workflow Automation** (formerly Integrify) into Kubernetes. The chart is published to GitHub Pages via the `helm/chart-releaser-action` on every push to `main`.

The single chart lives at `charts/integrify/` and is named `nutrient-workflow-automation` in `Chart.yaml`. The `appVersion` tracks the Nutrient Workflow Automation application release; `version` tracks chart changes — both should be bumped together on each release.

## Common Commands

```bash
# Lint the chart
helm lint charts/integrify/

# Render templates locally (requires a values file)
helm template release-name charts/integrify/ -f <values.yaml>

# Dry-run install against a live cluster
helm install -f <values.yaml> workflow-automation charts/integrify/ --dry-run

# Update chart dependencies (document-engine subchart)
helm dependency update charts/integrify/
```

`custom-values.yaml` is gitignored — use it for local testing values.

## Chart Architecture

Every Nutrient Workflow Automation microservice follows the same three-file pattern inside `charts/integrify/templates/`:

- `<service>-deployment.yaml` — Deployment, pulls image from `image.repo/<service>.repoName:release`, injects `globalEnv` + per-service `env`, mounts `integrifyconfig` ConfigMap at `/app/config`, and references the `integrify-secrets` Secret via `envFrom`.
- `<service>-service.yaml` — ClusterIP Service wiring container port.
- `<service>-hpa.yaml` — HorizontalPodAutoscaler scaling on memory utilization.

**Shared resources:**
- `configmap.yaml` — Renders `values.config` as `/app/config/config.yml` in every container.
- `secrets.yaml` — Encodes `values.secrets.*` (AWS creds, Redis connection, env token, reCAPTCHA key, credentials secret, document authoring license) into an `integrify-secrets` Secret consumed by all deployments via `envFrom`.
- `integrify-ingress.yaml` — Single Ingress routing all public paths (`/`, `/api/*`, `/workflow/*`, `/files`, etc.) to the correct backend services.

**Notable service behaviors:**
- `files` deployment uses `strategy: Recreate` and optionally mounts a PVC (`files.existingClaim`) at `/integrify/files` for local file storage. When not using a PVC, file storage relies on AWS S3 credentials in the Secret.
- `session-processor`, `task-processor`, and `config-processor` have no exposed port and receive no readiness/liveness probes.
- `scheduler` has a Service and BackendConfig but no `ports:` in its Deployment — probes are omitted for it too.
- `document-engine` is an optional subchart dependency (condition: `document-engine.enabled`) sourced from the PSPDFKit Helm repo.

**Service annotations** are conditional on `ingress.controller`:
- `alb` — renders `alb.ingress.kubernetes.io/healthcheck-path` on each Service
- `gce` / `gce-internal` — renders `cloud.google.com/backend-config` pointing to a per-service BackendConfig
- All others — no service annotations

**Readiness and liveness probes** are added to all deployments that have both a `port` and `healthcheck` value defined (16 services). Both probes hit the same `healthcheck` path.

## Ingress Controller Support

Three deployment environments are supported, configured via `ingress.controller` in values.yaml:

| Value | Environment | Notes |
|---|---|---|
| `nginx-f5` | On-premise (default) | F5 NGINX Ingress Controller; uses `nginx.org/*` annotations |
| `nginx-community` | On-premise (legacy) | ingress-nginx — **retired March 2026**, avoid for new deployments |
| `alb` | AWS | AWS Load Balancer Controller; uses `alb.ingress.kubernetes.io/*` annotations |
| `gce` | GKE | Google Cloud external HTTP(S) LB |
| `gce-internal` | GKE | Google Cloud internal HTTP(S) LB |

**`ingress.controller` vs `ingress.className`:** `controller` is used internally by the chart to conditionally render resources. `className` sets `ingressClassName` on the Ingress spec and should match the class configured on the controller (both F5 and ingress-nginx default to `nginx`; omit to preserve classless behaviour).

**`serviceType`:** Must be `NodePort` for GCE ingress; defaults to `ClusterIP` for nginx-f5 and ALB (ALB with `target-type: ip` also works with `ClusterIP`).

**BackendConfig resources** (`backendconfigs.yaml`) are rendered only for `gce` and `gce-internal`, creating one `cloud.google.com/v1 BackendConfig` per service with a health check configured from each service's `healthcheck` and `port` values.

**Legacy redirect path:** `/rest-service/files/stream` (routes to `redirect-v7-downloads` via the `use-annotation` port convention) is only rendered for `nginx-community` and `alb`. For ALB, the ingress.annotations block must include `alb.ingress.kubernetes.io/actions.redirect-v7-downloads` with a 301 redirect to `HTTPS://#{host}:#{port}/workflow/napi/files/download?#{query}` — see the ALB example in values.yaml.

## Versioning & Release

Releases are automated: merging to `main` triggers `helm/chart-releaser-action`, which packages the chart, creates a GitHub Release tagged `nutrient-workflow-automation-<chart-version>`, and updates the `gh-pages` branch index so `helm repo add` consumers receive the update.

When bumping versions, update both `version` (chart semver) and `appVersion` (app release string) in `charts/integrify/Chart.yaml`.
