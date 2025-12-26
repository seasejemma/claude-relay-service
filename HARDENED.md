# Hardened Fork

This is a hardened fork of [Wei-Shaw/claude-relay-service](https://github.com/Wei-Shaw/claude-relay-service).

## Changes from Upstream

- **Hardened Dockerfile**: Uses Docker Hardened Images (DHI) with Node 20 LTS base
- **Multi-arch builds**: Supports linux/amd64 and linux/arm64
- **Automated sync**: Syncs with upstream every 30 minutes
- **GHCR publishing**: Images published to `ghcr.io/seasejemma/claude-relay-service`

## Image Tags

- `ghcr.io/seasejemma/claude-relay-service:latest` - Latest release
- `ghcr.io/seasejemma/claude-relay-service:<version>` - Specific version (e.g., `1.1.241`)

## Security Features

- DHI base images (dhi.io/node:20-dev) - hardened Node.js runtime
- Multi-stage build minimizes attack surface
- Non-root user execution where possible
- Health checks enabled

## Usage

```bash
docker pull ghcr.io/seasejemma/claude-relay-service:latest
```

Or use with docker-compose:

```yaml
services:
  claude-relay:
    image: ghcr.io/seasejemma/claude-relay-service:latest
    ports:
      - "3000:3000"
    environment:
      - REDIS_HOST=redis
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs
```

## Requirements

For building locally with DHI images, Docker Hub credentials are required:

```bash
docker login dhi.io
```

## Upstream

This fork automatically syncs with upstream. Custom changes are maintained in:
- `Dockerfile.hardened`
- `.github/workflows/ghcr-hardened.yml`
- `.github/workflows/sync-upstream.yml`
- `docker-compose.override.yml`
- `HARDENED.md`
