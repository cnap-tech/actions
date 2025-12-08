# CNAP Reusable Actions

This directory contains reusable composite actions that can be used independently or as part of the main CNAP workflow.

## Available Actions

### `setup-buildkit`

Starts a BuildKit container for Docker builds.

**Usage:**
```yaml
- uses: ./.github/actions/setup-buildkit
```

**Requirements:**
- Docker must be available in the runner

**Outputs:**
- Sets `BUILDKIT_HOST` environment variable

---

### `generate-railpack-config`

Generates a `railpack.json` configuration file based on build settings.

**Usage:**
```yaml
- uses: ./.github/actions/generate-railpack-config
  with:
    build-apt-packages: 'curl wget'  # Optional
    runtime-apt-packages: 'nginx'    # Optional
    start-command: 'npm start'       # Optional
    build-context: './'              # Optional, default: './'
```

**Inputs:**
- `build-apt-packages` (optional): Space-separated list of apt packages needed during build
- `runtime-apt-packages` (optional): Space-separated list of apt packages needed at runtime
- `start-command` (optional): Override the detected start command
- `build-context` (optional): Directory containing application code (default: `./`)

**Note:** This action only generates the config file if at least one input is provided.

---

### `build-railpack`

Builds a Docker image using Railpack.

**Usage:**
```yaml
- uses: ./.github/actions/build-railpack
  id: build
  with:
    image-name: 'ghcr.io/owner/repo:tag'
    build-command: 'npm run build'   # Optional
    start-command: 'npm start'       # Optional
    build-context: './'              # Optional, default: './'
```

**Inputs:**
- `image-name` (required): Full Docker image name with tag
- `build-command` (optional): Override the detected build command
- `start-command` (optional): Override the detected start command (only used if no railpack.json exists)
- `build-context` (optional): Directory containing application code (default: `./`)

**Outputs:**
- `image`: The built Docker image name

**Requirements:**
- Railpack must be installed
- BuildKit must be set up (use `setup-buildkit` action)
- `BUILDKIT_HOST` environment variable must be set

---

### `notify-cnap`

Sends a build notification to the CNAP API.

**Usage:**
```yaml
- uses: ./.github/actions/notify-cnap
  with:
    image: 'ghcr.io/owner/repo:sha'
    image-tag: 'abc123def'
```

**Inputs:**
- `image` (required): Full Docker image name
- `image-tag` (required): Image tag (usually commit SHA)
- `notify-url` (optional): CNAP API URL (default: `https://dash.cnap.tech/api/github/build-notify`)

**Requirements:**
- OIDC token must be available via `ACTIONS_ID_TOKEN_REQUEST_URL` and `ACTIONS_ID_TOKEN_REQUEST_TOKEN` environment variables
- Workflow must have `id-token: write` permission

---

### `stop-buildkit`

Stops and cleans up the BuildKit container.

**Usage:**
```yaml
- uses: ./.github/actions/stop-buildkit
  if: always()  # Recommended to ensure cleanup even on failure
```

**Requirements:**
- BuildKit container must be running (started by `setup-buildkit`)

---

## Example: Using Actions Independently

You can use these actions in your own workflows:

```yaml
name: Custom Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
      
      - uses: docker/setup-buildx-action@v3
      
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - run: curl -sSL https://railpack.com/install.sh | bash
      
      - uses: cnap-tech/actions/.github/actions/setup-buildkit@main
      
      - uses: cnap-tech/actions/.github/actions/build-railpack@main
        id: build
        with:
          image-name: ghcr.io/${{ github.repository }}:${{ github.sha }}
      
      - run: docker push ${{ steps.build.outputs.image }}
      
      - uses: cnap-tech/actions/.github/actions/notify-cnap@main
        with:
          image: ${{ steps.build.outputs.image }}
          image-tag: ${{ github.sha }}
      
      - uses: cnap-tech/actions/.github/actions/stop-buildkit@main
        if: always()
```

## Example: Using Actions Locally

If you're referencing these actions from the same repository:

```yaml
- uses: ./.github/actions/build-railpack
  with:
    image-name: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

## Benefits

- **Modularity**: Each action has a single responsibility
- **Reusability**: Actions can be used independently or together
- **Maintainability**: Easier to update and test individual components
- **Flexibility**: Developers can use only the actions they need

