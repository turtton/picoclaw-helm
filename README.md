# PicoClaw Helm Chart

Helm chart for deploying PicoClaw, a lightweight AI assistant, on Kubernetes.

[Japanese version](README.ja.md)

## Prerequisites

- Kubernetes cluster
- Helm 3+

## Installation

Current version:
- Chart version: `0.1.11`
- App version: `v0.2.5`

Add the Helm repository and install the chart:

```bash
helm repo add picoclaw https://turtton.github.io/picoclaw-helm
helm repo update
helm install my-picoclaw picoclaw/picoclaw --version 0.1.11
```

## Configuration

The following table lists the most important configuration parameters for the PicoClaw chart.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | PicoClaw image repository | `docker.io/sipeed/picoclaw` |
| `image.tag` | PicoClaw image tag (defaults to `appVersion`) | `""` |
| `config.agents` | Agent configuration (defaults, list) | See `values.yaml` |
| `config.model_list` | Model provider configuration | `[]` |
| `config.channels` | Communication channel configuration (discord, etc) | See `values.yaml` |
| `securityConfig` | Inline sensitive data (API keys, tokens) for `.security.yml` | `{}` |
| `securitySecret.existingSecret` | Pre-existing Secret containing `.security.yml` | `""` |
| `workspaceFiles` | Files to inject into `PICOCLAW_HOME/workspace/` | `{}` |
| `skillFiles` | Skill definitions to inject | `{}` |
| `persistence.enabled` | Enable persistent volume for `PICOCLAW_HOME` | `true` |
| `persistence.size` | Size of the persistent volume | `5Gi` |
| `launcher.enabled` | Enable the optional WebUI launcher sidecar | `false` |
| `launcher.port` | Port for the launcher WebUI | `18800` |
| `copilotCli.enabled` | Enable the GitHub Copilot CLI sidecar (requires `copilotCli.tokenSecret.name`) | `false` |
| `service.type` | Kubernetes Service type | `ClusterIP` |
| `service.port` | Gateway port (default `18790`; do not change without chart modification) | `18790` |
| `resources` | Pod resource requests and limits | `{}` |
| `nodeSelector` | Node selection labels | `{}` |
| `tolerations` | Pod tolerations | `[]` |
| `affinity` | Pod affinity rules | `{}` |

## Security Configuration

If you need to store sensitive data like API keys or bot tokens, you should provide a `.security.yml` file. PicoClaw can use this file for various providers. You have two options for providing this:

1. **`securityConfig`**: Define the security configuration directly in your `values.yaml`. The chart will create a Secret for you.
2. **`securitySecret.existingSecret`**: Use an existing Secret that contains the `.security.yml` content. The Secret must have a key (default `security.yml`) containing the YAML data.

## Workspace and Skills

You can inject custom workspace and skill files into the PicoClaw instance at startup.

- **`workspaceFiles`**: A map where keys are filenames and values are the content. These files are copied to `/root/.picoclaw/workspace/`.
- **`skillFiles`**: Injects skill definitions into the container at startup. Each entry is copied from its ConfigMap to `/root/.picoclaw/workspace/skills/<name>/`.

Example:
```yaml
skillFiles:
  my-skill:
    configMapName: my-skill-cm
    items:
      - key: SKILL.md
        path: SKILL.md
```

## Launcher WebUI

PicoClaw can run an optional WebUI by enabling the launcher sidecar:

```yaml
launcher:
  enabled: true
```

The launcher provides a web interface that proxies requests to the PicoClaw gateway. It uses a separate image tag (`launcher`) by default.

## Copilot CLI

The chart can run a GitHub Copilot CLI sidecar for AI-assisted shell access:

```yaml
copilotCli:
  enabled: true
  tokenSecret:
    name: my-copilot-token
```

The sidecar requires a pre-existing Secret containing a GitHub Copilot token. See `values.yaml` for full options.

## Architecture

- `PICOCLAW_HOME` is fixed at `/root/.picoclaw` and backed by a PersistentVolumeClaim (or `emptyDir` when `persistence.enabled=false`)
- An init container copies `config.json`, `.security.yml` (if configured), workspace files, and skill files into the volume before the main container starts
- The main container runs `picoclaw gateway -E`
- Deployment strategy is `Recreate` (single replica assumed)
- Only `config.json` changes (via `config.*` values) trigger automatic redeployment; updates to `securityConfig`, `workspaceFiles`, or `skillFiles` require a manual pod restart

## Uninstalling the Chart

To uninstall the `my-picoclaw` release:

```bash
helm uninstall my-picoclaw
```

## Upstream Project

For more details about PicoClaw, visit the [official repository](https://github.com/sipeed/picoclaw).
