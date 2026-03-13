# Mattermost Docker Upgrade Guide

Steps to upgrade Mattermost hosted on AWS Lightsail (Docker deployment, no NGINX override).

## Before You Start

1. **Take a Lightsail snapshot** — Lightsail console → your instance → Snapshots → Create snapshot. This is your full rollback point.

## Upgrade Steps

### 1. SSH into your Lightsail instance and navigate to the docker folder

```bash
cd ~/docker
```

### 2. Stop the running containers

```bash
docker compose -f docker-compose.yml down
```

### 3. Update the version tag

Open the `.env` file:

```bash
nano .env
```

Update `MATTERMOST_IMAGE_TAG` to the target version, e.g.:

```
MATTERMOST_IMAGE_TAG=release-10.11.12
```

- **ESR (Extended Support Release):** safer for production, longer support window
- **Latest stable:** more features, updated more frequently
- Check available tags at: https://hub.docker.com/r/mattermost/mattermost-team-edition/tags

### 4. Pull the new image and redeploy

```bash
docker compose -f docker-compose.yml up -d
```

### 5. Verify it's running

```bash
docker ps
```

---

## Important Notes

- **Never skip major versions.** If you're multiple versions behind, upgrade through ESRs one at a time (e.g. 7.x → 8.1 → 9.5 → 10.x). Skipping versions can break database migrations.
- **v6.0** has a heavy DB migration that can take ~11 minutes and lock tables — plan a maintenance window if crossing that version.
- Check your current version before upgrading: `grep MATTERMOST_IMAGE_TAG .env`

## Current ESR (as of March 2026)

| Version | Type | Notes |
|---|---|---|
| 10.11.12 | ESR | Recommended for production |
| 11.4.2 | Latest stable | Latest features |
