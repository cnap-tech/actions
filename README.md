# Reusable GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows to simplify common CI/CD tasks.

## Available Workflows

### Reusable Deploy Workflow

A reusable workflow designed to simplify deployments across multiple repositories. This workflow allows you to standardize your deployment process and automatically inherit all secrets from the calling repository.

**Location:** `.github/workflows/reusable-deploy.yml`

#### Features

- Inherit all secrets automatically from the caller repository (no need to pass secrets individually)
- Configurable image tag for deployments
- Configurable deployment environment (dev, staging, production)
- Built-in Docker authentication with GitHub Container Registry

#### Usage

##### Option 1: Inherit All Secrets (Recommended)

This is the simplest approach - all secrets from your repository are automatically available to the reusable workflow:

```yaml
name: Deploy Application

on:
  push:
    branches:
      - main

jobs:
  deploy:
    uses: cnap-tech/workflows/.github/workflows/reusable-deploy.yml@main
    with:
      image_tag: latest
      environment: production
    secrets: inherit  # Automatically inherits ALL secrets from the caller repo
```

##### Option 2: Pass Specific Secrets

If you prefer to pass secrets explicitly:

```yaml
name: Deploy Application

on:
  push:
    branches:
      - main

jobs:
  deploy:
    uses: cnap-tech/workflows/.github/workflows/reusable-deploy.yml@main
    with:
      image_tag: latest
      environment: production
    secrets:
      GHCR_PAT: ${{ secrets.GHCR_PAT }}  # Passing secret explicitly
```

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `image_tag` | Docker image tag to deploy | No | `latest` |
| `environment` | Deployment environment (e.g., dev, staging, production) | No | `production` |

#### Secrets

| Secret | Description | Required |
|--------|-------------|----------|
| `GHCR_PAT` | GitHub Container Registry Personal Access Token | No |

When using `secrets: inherit`, all secrets from your repository are automatically available, including `GHCR_PAT` and any other secrets you've configured.

#### Example: Multi-Environment Deployment

```yaml
name: Multi-Environment Deploy

on:
  push:
    branches:
      - main
      - staging
      - develop

jobs:
  deploy-production:
    if: github.ref == 'refs/heads/main'
    uses: cnap-tech/workflows/.github/workflows/reusable-deploy.yml@main
    with:
      image_tag: ${{ github.sha }}
      environment: production
    secrets: inherit

  deploy-staging:
    if: github.ref == 'refs/heads/staging'
    uses: cnap-tech/workflows/.github/workflows/reusable-deploy.yml@main
    with:
      image_tag: ${{ github.sha }}
      environment: staging
    secrets: inherit

  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    uses: cnap-tech/workflows/.github/workflows/reusable-deploy.yml@main
    with:
      image_tag: latest
      environment: dev
    secrets: inherit
```

## Benefits of Using Reusable Workflows

- **Consistency:** Ensures all your repositories use the same deployment process
- **Maintainability:** Update the workflow once, and all repositories benefit
- **Security:** Centralize secret handling and reduce duplication
- **Simplicity:** With `secrets: inherit`, no need to pass secrets individually

## Contributing

Feel free to submit issues or pull requests to improve these workflows.