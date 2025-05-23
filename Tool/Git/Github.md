
# Basic Go Workflow

```yaml
name: Docker Build 

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    - name: Build the Docker image
      run: docker build . --file Dockerfile --tag my-image-name:$(date +%s)
```

# Base for a release Go Workflow

```yaml
name: Docker Release

on:
  push:
    # Publish semver tags as releases.
    tags: [ 'v*.*.*' ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      # This is used to complete the identity challenge
      # with sigstore/fulcio
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # Install the cosign tool
      - name: Install cosign
        uses: sigstore/cosign-installer@v3.8.0
        with:
          cosign-release: 'v2.4.2'

      # Set up BuildKit Docker container builder to be able to build
      # multi-platform images and export cache
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3.9.0

      # Login against a Docker registr
      - name: Log into registry ${{ env.REGISTRY }}
        uses: docker/login-action@v3.3.0
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Extract metadata (tags, labels) for Docker
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5.6.1
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      # Build and push Docker image with Buildx 
      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@v6.13.0
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

# Dependabot Config

```
.github/dependabot.yml
```

```yaml
version: 2
updates:
  # Enable verison updates for Go
  - package-ecosystem: "gomod" 
    # Look for gomod in root directory
    directory: "/"
    # Check for updates once a week
    schedule:
      interval: "weekly"

  # Enable version updates for Docker
  - package-ecosystem: "docker"
    # Look for a `Dockerfile` in the `root` directory
    directory: "/"
    # Check for updates once a month
    schedule:
      interval: "monthly"
```