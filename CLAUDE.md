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
- `session-processor` and `task-processor` have no per-service env (only `globalEnv`).
- `document-engine` is an optional subchart dependency (condition: `document-engine.enabled`) sourced from the PSPDFKit Helm repo.

## Versioning & Release

Releases are automated: merging to `main` triggers `helm/chart-releaser-action`, which packages the chart, creates a GitHub Release tagged `nutrient-workflow-automation-<chart-version>`, and updates the `gh-pages` branch index so `helm repo add` consumers receive the update.

When bumping versions, update both `version` (chart semver) and `appVersion` (app release string) in `charts/integrify/Chart.yaml`.
