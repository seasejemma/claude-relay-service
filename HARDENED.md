# Claude Relay Service (Hardened Fork)

Hardened fork of [Wei-Shaw/claude-relay-service](https://github.com/Wei-Shaw/claude-relay-service) with DHI (Docker Hardened Images) runtime.

## Image

```
ghcr.io/seasejemma/claude-relay-service:latest
ghcr.io/seasejemma/claude-relay-service:<version>
```

Note: Image tags use semver without `v` prefix (e.g., `1.1.249`, not `v1.1.249`).

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ghcr-hardened.yml` | Tag push `v*`, manual | Build multi-arch image to GHCR |
| `sync-upstream.yml` | Every 30 min, manual | Sync from upstream, push new tags |

## Manual Triggers

```bash
# Trigger upstream sync
gh workflow run sync-upstream.yml --repo seasejemma/claude-relay-service

# Trigger build with custom tag
gh workflow run ghcr-hardened.yml --repo seasejemma/claude-relay-service -f tag=v1.2.3
```

## Key Files (Custom)

- `Dockerfile.hardened` - DHI-based hardened image (Node.js)
- `.github/workflows/ghcr-hardened.yml` - GHCR build workflow
- `.github/workflows/sync-upstream.yml` - Upstream sync workflow
- `docker-compose.override.yml` - Local customizations
- `HARDENED.md` - This file

## Topics

`tokeninfra`, `adt`

---

# Hardened Fork Methodology

This section documents the methodology for creating hardened Docker image builds for forked repositories.

## Overview

The methodology creates automated pipelines that:
1. Fork upstream repos to a personal GitHub account
2. Build hardened Docker images using minimal/distroless base images
3. Publish multi-arch images to GHCR
4. Automatically sync with upstream and build new releases

## Base Image Selection

| Project Type | Build Stage | Runtime Stage |
|--------------|-------------|---------------|
| Go | `golang:X.XX-alpine` | `gcr.io/distroless/static:nonroot` |
| Node.js | `dhi.io/node:XX-dev` | `dhi.io/node:XX` |

- **Go projects**: Use distroless static image (no shell, minimal attack surface)
- **Node.js projects**: Use DHI (Docker Hardened Images) which require `DOCKER_USERNAME`/`DOCKER_PASSWORD` secrets

## Workflow Structure

### sync-upstream.yml

Runs every 30 minutes to:
1. Fetch upstream changes and tags
2. Merge upstream/main into fork's main
3. Remove conflicting upstream workflows
4. Push new tags to fork
5. Check if image exists for latest tag
6. Trigger build if image missing

**Key robustness fixes:**
- `concurrency: group: sync-upstream` - Prevents duplicate runs
- `fetch-tags: true` - Ensures tags are fetched
- `--force` on tag fetch - Handles upstream force-pushed tags
- GHCR login before `docker manifest inspect` - Required for authentication
- Lowercase repo name conversion - GHCR requires lowercase (`github.repository` may return mixed case)
- Correct tag format for image check - Match how images are actually tagged

### ghcr-hardened.yml

Triggered by:
- Tag push matching `v*`
- Manual workflow dispatch with optional tag input

Features:
- Multi-arch builds (linux/amd64, linux/arm64)
- GitHub Actions cache for faster builds
- Automatic GitHub Release creation
- Concurrency groups to prevent duplicate builds

## Image Tagging Strategy

Different repos may use different tag formats:

| Repo | Git Tag | Image Tag | Check Logic |
|------|---------|-----------|-------------|
| claude-relay-service | `v1.1.249` | `1.1.249` | Strip `v` prefix |
| CLIProxyAPIPlus | `v6.6.63-0` | `v6.6.63-0` | Keep full tag |

The sync workflow must match the image tagging strategy used by the build workflow.

## Required Secrets

| Secret | Purpose | Required For |
|--------|---------|--------------|
| `GITHUB_TOKEN` | Auto-provided, GHCR auth, workflow dispatch | All |
| `DOCKER_USERNAME` | DHI registry login | Node.js projects |
| `DOCKER_PASSWORD` | DHI registry login | Node.js projects |

## Repository Setup Checklist

1. Fork upstream repo to GitHub account
2. Create `Dockerfile.hardened` with appropriate base images
3. Create `.github/workflows/ghcr-hardened.yml`
4. Create `.github/workflows/sync-upstream.yml`
5. Create `docker-compose.override.yml` for local customizations
6. Create `HARDENED.md` documentation
7. Add secrets if using DHI images
8. Add topics: `tokeninfra`, `adt`
9. Single commit with author: `jemma <jemma@jane.doe>`

## docker-compose.override.yml Pattern

```yaml
services:
  <service-name>:
    container_name: <short-name>
    networks:
      - webproxy

networks:
  webproxy:
    external: true
```

## Common Issues and Fixes

### Build triggers every 30 minutes even when no new tags

**Causes:**
1. Missing GHCR login - `docker manifest inspect` requires authentication
2. Case sensitivity - GHCR requires lowercase image names
3. Tag format mismatch - Image tag format must match what build workflow produces

**Fix:** Add GHCR login step and use lowercase repo name:
```yaml
- name: Log in to GHCR for manifest check
  run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

- name: Push new tags and trigger build
  run: |
    REPO_LOWER=$(echo "${{ github.repository }}" | tr '[:upper:]' '[:lower:]')
    IMAGE="ghcr.io/${REPO_LOWER}:${TAG}"
    docker manifest inspect "$IMAGE"
```

### Upstream force-pushed tags not synced

**Fix:** Add `--force` to tag fetch:
```yaml
git fetch upstream --tags --force
```

### Duplicate workflow runs

**Fix:** Add concurrency groups:
```yaml
concurrency:
  group: sync-upstream
  cancel-in-progress: true
```
