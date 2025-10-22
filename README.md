# CNAP Workflows

Reusable GitHub Actions workflows for building and deploying applications with CNAP.

## 🚀 Build Workflow

The `build.yml` workflow provides automated building using Railpack (successor to Nixpacks) and pushes images to GitHub Container Registry.

### Features

- **Automatic Detection**: Railpack automatically detects your application type and build configuration
- **Customizable**: Override detected commands and add custom packages
- **Minimal Configuration**: Works out of the box with sensible defaults
- **Secret Inheritance**: Automatically uses `GITHUB_TOKEN` for registry authentication

### Basic Usage

Add this workflow to your repository at `.github/workflows/cnap.yml`:

```yaml
name: CNAP Build & Deploy
on:
  push:
    branches: [main]
  workflow_call:

jobs:
  build:
    uses: cnap-tech/workflows/.github/workflows/build.yml@main
    secrets: inherit
```

### Advanced Usage

Override build settings when needed:

```yaml
name: CNAP Build & Deploy
on:
  push:
    branches: [main]
  workflow_call:

jobs:
  build:
    uses: cnap-tech/workflows/.github/workflows/build.yml@main
    with:
      build-context: './backend'
      build-command: 'npm run build'
      start-command: 'npm start'
      install-command: 'npm ci'
      build-apt-packages: 'python3 make g++'
      runtime-apt-packages: 'ca-certificates'
    secrets: inherit
```

## 📋 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `build-context` | Directory containing your application code | No | `./` |
| `build-command` | Override the detected build command | No | Auto-detected |
| `start-command` | Override the detected start command | No | Auto-detected |
| `install-command` | Override the detected install command | No | Auto-detected |
| `build-apt-packages` | Additional apt packages needed during build | No | None |
| `runtime-apt-packages` | Additional apt packages needed at runtime | No | None |

## 🔐 Secrets

The workflow uses `secrets: inherit` to automatically access:
- `GITHUB_TOKEN` - For pushing to GitHub Container Registry

## 📦 What Gets Built

The workflow:
1. Checks out your code
2. Uses Railpack to detect and build your application
3. Creates a Docker image tagged with the commit SHA
4. Pushes to `ghcr.io/your-org/your-repo:commit-sha`
5. Notifies CNAP when the build is ready

## 🛠️ Supported Languages

Railpack automatically detects and builds:
- Node.js / TypeScript
- Python
- Go
- Rust
- Ruby
- PHP
- Java / Kotlin
- .NET
- And more...

## 📖 Examples

### Monorepo with Custom Context

```yaml
jobs:
  build-api:
    uses: cnap-tech/workflows/.github/workflows/build.yml@main
    with:
      build-context: './apps/api'
    secrets: inherit
  
  build-web:
    uses: cnap-tech/workflows/.github/workflows/build.yml@main
    with:
      build-context: './apps/web'
    secrets: inherit
```

### Python App with System Dependencies

```yaml
jobs:
  build:
    uses: cnap-tech/workflows/.github/workflows/build.yml@main
    with:
      build-apt-packages: 'postgresql-dev libxml2-dev'
      runtime-apt-packages: 'postgresql-client'
    secrets: inherit
```

## 🔗 Learn More

- [Railpack Documentation](https://railpack.app)
- [CNAP Documentation](https://cnap.tech/docs)
- [GitHub Actions Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

## 📝 License

MIT
