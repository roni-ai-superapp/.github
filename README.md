# roni-ai-superapp/.github

Shared GitHub Actions and reusable workflows for the roni-ai-superapp organization.

## deploy-v1 Contract

The `deploy-v1` reusable workflow provides a standardized Railway deployment primitive for all services.

### Usage

```yaml
# .github/workflows/deploy-dev.yml
name: Deploy to Dev

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: roni-ai-superapp/.github/.github/workflows/railway-deploy.yml@deploy-v1
    with:
      railway_service: my-service
      railway_environment: dev
      github_environment: dev
      service_url: https://my-service-dev.up.railway.app
      commit_sha: ${{ github.sha }}
    secrets: inherit
```

### Health Contract

All services must expose a health endpoint returning:

```json
{
  "service": "my-service",
  "environment": "dev",
  "status": "ok",
  "git_sha": "abc123def456789012345678901234567890abcd",
  "commit": "abc123def456",
  "release_tag": "v1.2.3"
}
```

### Required Secrets

| Scope | Secret | Description |
|-------|--------|-------------|
| Repository | `RAILWAY_PROJECT_ID` | Railway platform project ID |
| Environment | `RAILWAY_TOKEN` | Per-environment Railway token |

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `railway_service` | Yes | - | Service name in Railway |
| `railway_environment` | Yes | - | Railway environment (dev, staging, production) |
| `github_environment` | Yes | - | GitHub environment for secrets |
| `service_url` | Yes | - | Public URL for health checks |
| `health_path` | No | `/` | Health endpoint path |
| `commit_sha` | Yes | - | Full 40-char commit SHA |
| `release_tag` | No | - | Release tag for production |
| `timeout_seconds` | No | 600 | Verification timeout |
| `railway_cli_version` | No | 4.11.0 | Pinned Railway CLI version |

## Versioning

- `deploy-v1`: Current stable version
- Future breaking changes will be `deploy-v2`, etc.

## Composite Actions

### railway-setup

Shared action for Railway CLI installation and project configuration.

```yaml
- uses: roni-ai-superapp/.github/.github/actions/railway-setup@deploy-v1
  with:
    cli_version: "4.11.0"
    project_id: ${{ secrets.RAILWAY_PROJECT_ID }}
```
