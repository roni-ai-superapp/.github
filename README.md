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
    uses: roni-ai-superapp/.github/.github/workflows/railway-deploy.yml@deploy-v1.1
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
  "ok": true,
  "db_ok": true,
  "git_sha": "abc123def456789012345678901234567890abcd",
  "commit": "abc123def456",
  "release_tag": "v1.2.3"
}
```

**Required fields for verification:**
- `git_sha`: Full 40-character hex SHA (or "unknown")
- `commit`: Short 12-character SHA
- `ok` or `status`: Must be `true` or `"ok"` for deployment to pass
- `db_ok`: Must be `true` (defaults to true if not present)

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

## Tag Protection (Required Setup)

### In this repo (`.github`)

Protect `deploy-v*` tags to prevent accidental deletion/retagging:

```bash
# Create ruleset via GitHub UI:
# Settings → Rules → Rulesets → New ruleset
# - Name: "Protect deploy versions"
# - Target: Tags matching "deploy-v*"
# - Rules: Restrict deletions, Restrict updates
```

### In service repos (parser, accounting-db, etc.)

Protect `v*` release tags with bot bypass for release-please:

```bash
# Settings → Rules → Rulesets → New ruleset
# - Name: "Protect release tags"
# - Target: Tags matching "v*"
# - Rules: Restrict deletions, Restrict updates
# - Bypass: Add the PAT identity used by RELEASE_PLEASE_TOKEN
```

## Pre-First-Deploy Checklist

Before deploying a new service for the first time:

1. **Create Railway service** in platform project (dev, staging, production environments)

2. **Generate Railway tokens** - one per environment, scoped appropriately

3. **Create GitHub environments** (plain names, no URL encoding needed):
   ```bash
   gh api -X PUT repos/roni-ai-superapp/{service}/environments/dev
   gh api -X PUT repos/roni-ai-superapp/{service}/environments/staging
   gh api -X PUT repos/roni-ai-superapp/{service}/environments/production
   ```

4. **Configure production environment protection** (required reviewers) BEFORE first run

5. **Set GitHub secrets**:
   - Repository: `RAILWAY_PROJECT_ID`, `RELEASE_PLEASE_TOKEN`
   - Per-environment: `RAILWAY_TOKEN`

6. **Set Railway environment variables**:
   - `RAILWAY_ENVIRONMENT_NAME`: dev/staging/production

7. **Add tag protection rules** (see above)

8. **Update service health endpoint** to return deploy-v1 contract fields

## Composite Actions

### railway-setup

Shared action for Railway CLI installation and project configuration.

```yaml
- uses: roni-ai-superapp/.github/.github/actions/railway-setup@deploy-v1.1
  with:
    cli_version: "4.11.0"
    project_id: ${{ secrets.RAILWAY_PROJECT_ID }}
```
