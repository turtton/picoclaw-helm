# AGENTS.md

## Overview

Helm chart repository for [PicoClaw](https://github.com/sipeed/picoclaw) — lightweight AI assistant for Kubernetes. Single chart, no tests, no linting configured.

## Structure

```
chart/picoclaw/          # The only chart
  Chart.yaml             # apiVersion: v2, type: application
  values.yaml            # All configurable values
  templates/
    _helpers.tpl          # Name/label helpers + picoHome (/root/.picoclaw)
    deployment.yaml       # Main gateway + optional launcher & copilot-cli sidecars
    configmap.yaml        # Generates config.json from .Values.config.*
    workspace-configmap.yaml  # Workspace files injected via init container
    secret.yaml           # .security.yml from securityConfig
    service.yaml          # Gateway + optional launcher ports
    pvc.yaml              # Persistence for PICOCLAW_HOME
    extra-manifests.yaml  # Arbitrary extra K8s manifests
```

## Key Architecture Details

- **PICOCLAW_HOME** is hardcoded to `/root/.picoclaw` in `_helpers.tpl` (`picoclaw.picoHome`)
- **Init container** copies config.json, .security.yml, workspace files, and skill files into the PVC before the main container starts
- **Main container** runs `picoclaw gateway -E` (bypasses entrypoint.sh)
- **Deployment strategy** is `Recreate` (not RollingUpdate) — single replica assumed
- **Config checksum** annotation on pod triggers redeployment on config changes
- **Security secrets** have two paths: inline `securityConfig` (chart-managed Secret) or `securitySecret.existingSecret` (pre-existing Secret)
- **Launcher sidecar** (WebUI) and **copilot-cli sidecar** are optional, gated by `.Values.launcher.enabled` / `.Values.copilotCli.enabled`

## Versioning

- `Chart.yaml` has both `version` (chart) and `appVersion` (PicoClaw image tag)
- Bump `version` for any chart change; bump `appVersion` when targeting a new PicoClaw release
- Image tag defaults to `appVersion` unless overridden via `image.tag`

## Release Process

- Push to `main` with changes under `chart/picoclaw/` or `.github/workflows/release.yml` triggers GitHub Actions
- CI uses [chart-releaser-action](https://github.com/helm/chart-releaser-action) v1.7.0 to package and publish to `gh-pages` branch
- `charts_dir: chart` — the action looks for charts one level under `chart/`

## Development Commands

```sh
# Lint (requires helm CLI)
helm lint chart/picoclaw

# Template render (dry-run)
helm template my-release chart/picoclaw -f my-values.yaml

# Install locally
helm install my-release chart/picoclaw -f my-values.yaml
```

No test suite, no pre-commit hooks, no Makefile.

## Conventions

- Commit messages follow `type: description` format (`feat:`, `fix:`, `chore:`, `ci:`)
- No README in repo — values.yaml comments are the primary documentation
- `.gitignore` excludes `.github/*` except `.github/workflows/`
