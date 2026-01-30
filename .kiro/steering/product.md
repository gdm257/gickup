# Product Overview

Gickup is a CLI tool for mirroring Git repositories between version control systems. It clones or mirrors repositories from source hosting platforms to destination targets, enabling backup and redundancy across providers.

## Core Capabilities

- **Multi-platform Repository Backup**: Pull from 8+ Git hosting services (GitHub, GitLab, Gitea, Gogs, Bitbucket, OneDev, Sourcehut, generic)
- **Flexible Destination Targets**: Push to multiple destinations (GitHub, GitLab, Gitea, Gogs, OneDev, Sourcehut, local filesystem, S3, Azure Blob)
- **Filtering & Selection**: Filter repos by user, org, language, stars, activity, archived/forks status
- **Scheduling & Automation**: Cron-based scheduling for recurring backups with config hot-reload
- **Observability**: Prometheus metrics, structured logging, and notification integrations (ntfy, Gotify, Apprise, heartbeat)

## Target Use Cases

- Backup repositories from a primary platform to a secondary one
- Migrate repositories between hosting providers
- Maintain offline mirrors of important repositories
- Multi-cloud repository redundancy

## Value Proposition

Single configuration-driven tool that handles complex multi-source to multi-destination backup scenarios without custom scripts. Supports both one-time backups and scheduled mirroring with LFS support, wiki backup, and issue tracking export.

---

_Focus on patterns and purpose, not exhaustive feature lists_
