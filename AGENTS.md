# AGENTS.md - Hardened Fork Customizations

This branch contains hardened Docker build customizations for the claude-relay-service fork.

## Overview

Hardened fork of [Wei-Shaw/claude-relay-service](https://github.com/Wei-Shaw/claude-relay-service) with:
- Docker Hardened Images (DHI) base
- Multi-arch builds (amd64/arm64)
- Automated upstream sync
- GHCR publishing

## Image

```bash
docker pull ghcr.io/seasejemma/claude-relay-service:latest
docker pull ghcr.io/seasejemma/claude-relay-service:<version>
```

## Custom Files

| File | Purpose |
|------|---------|
| `Dockerfile.hardened` | Multi-stage build using `dhi.io/node:20-dev` |
| `.github/workflows/ghcr-hardened.yml` | Build workflow for GHCR |
| `.github/workflows/sync-upstream.yml` | Upstream sync (every 30 min) |
| `docker-compose.override.yml` | Local customizations (crs, webproxy) |
| `HARDENED.md` | Fork documentation |

## Workflows

### ghcr-hardened.yml
- **Triggers**: Tag push `v*`, manual dispatch with tag input
- **Actions**: Build multi-arch image, push to GHCR, create GitHub Release
- **Secrets**: `DOCKER_USERNAME`, `DOCKER_PASSWORD` (for DHI access)

### sync-upstream.yml
- **Triggers**: Cron `*/30 * * * *`, manual dispatch
- **Actions**: 
  1. Merge upstream/main
  2. Remove `auto-release-pipeline.yml` (prevents duplicate commits)
  3. Push new tags
  4. Check if image exists in GHCR
  5. Trigger build if image missing

## Manual Commands

```bash
# Trigger upstream sync
gh workflow run sync-upstream.yml -R seasejemma/claude-relay-service

# Trigger build for specific tag
gh workflow run ghcr-hardened.yml -R seasejemma/claude-relay-service -f tag=v1.1.249

# Create release manually
gh release create v1.1.249 -R seasejemma/claude-relay-service --title "Release 1.1.249" --notes "Hardened build"
```

## Backup Locations

- **Branch**: `hardened-customizations` (this branch)
- **Local**: `/Users/liuwen/work/nexus/safekeep/claude-relay-service-hardened/`

## Git Identity

```
Username: jemma
Email: jemma@jane.doe
```

Configured via `~/.gitconfig-github-seasejemma` with `includeIf` pattern.
