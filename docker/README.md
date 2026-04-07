# Docker Services — Homelab (Debian VM)

Self-hosted service stack running on a Debian VM, managed via Docker Compose. **26 containers**, all running 24/7.

## Service Overview

| Category | Service | Image | Port | Purpose |
|----------|---------|-------|------|---------|
| **Management** | Portainer | `portainer-ce` | host | Docker GUI management |
| **Management** | Watchtower | `watchtower` | — | Auto-update containers (Sat 2am) |
| **Management** | Dozzle | `dozzle` | 8888 | Real-time Docker log viewer |
| **Management** | Prunemate | `prunemate` | 7676 | Container image cleanup scheduler |
| **Dashboard** | Homer | `b4bz/homer` | 3000 | Homelab start page / service index |
| **Media** | Jellyfin | `linuxserver/jellyfin` | host | Media server — movies & TV |
| **Photos** | Immich | `immich-server` | 2283 | Self-hosted Google Photos alternative |
| **Photos** | Immich ML | `immich-machine-learning` | — | Face recognition & CLIP search |
| **AI / LLM** | Open WebUI | `open-webui` | 3006 | Web UI for Ollama LLM (local + OpenRouter) |
| **AI / LLM** | Mirofish | `mirofish` | 3010/5001 | AI document & image analysis |
| **Automation** | n8n | `n8nio/n8n` | 8525 | Visual workflow automation |
| **Security** | Vaultwarden | `vaultwarden/server` | 9091 | Self-hosted Bitwarden password manager |
| **Finance** | Firefly III | `fireflyiii/core` | 8212 | Personal finance & budgeting |
| **Bookmarks** | Linkwarden | `linkwarden` | 3099 | Bookmark manager with full-page archiving |
| **DNS** | AdGuard Home | `adguardhome` | host | Network-wide DNS ad blocker |
| **File Sync** | Syncthing | `linuxserver/syncthing` | host | Peer-to-peer file synchronization |
| **Backup** | Duplicati | `linuxserver/duplicati` | 8200 | Encrypted scheduled backups |
| **Databases** | PostgreSQL (pgvector) | `ankane/pgvector` | — | n8n database with vector extension |
| **Databases** | MariaDB | `mariadb:11.4` | — | General-purpose SQL database |
| **Databases** | Immich Postgres | `immich-app/postgres` | — | Immich with pgvector + pgvectors |

## Architecture

```
Debian VM (Docker host)
│
├── Homer :3000           ← Start page linking all services
│
├── Media Stack
│   └── Jellyfin          ← Movies/TV (host network for DLNA)
│
├── Photo Stack
│   ├── Immich :2283      ← Photo sync from phone
│   ├── Immich ML         ← On-device face recognition + CLIP embeddings
│   ├── Immich Postgres   ← pgvector database
│   └── Immich Redis      ← Job queue
│
├── AI Stack
│   ├── Open WebUI :3006  ← Connects to Ollama (LXC 101) + OpenRouter
│   └── Mirofish :3010    ← Document/image AI analysis
│
├── Automation Stack
│   ├── n8n :8525         ← Workflow automation (connects to HA, Immich, etc.)
│   ├── n8n Postgres      ← pgvector DB for n8n
│   └── n8n Redis         ← n8n job queue
│
├── Self-Hosted Apps
│   ├── Vaultwarden :9091 ← Password manager (Bitwarden-compatible)
│   ├── Firefly III :8212 ← Personal finance tracking
│   └── Linkwarden :3099  ← Bookmark archiving + search
│
├── Network
│   └── AdGuard Home      ← DNS server for entire LAN, blocks ads/trackers
│
└── Maintenance
    ├── Syncthing         ← Config/data sync between machines
    ├── Duplicati :8200   ← Encrypted backups
    ├── Watchtower        ← Weekly auto-update of all images
    ├── Dozzle :8888      ← Log monitoring
    └── Prunemate :7676   ← Scheduled image pruning
```

## Notable Technical Details

- **Immich ML** runs on-device face recognition and CLIP semantic image search — no cloud
- **Open WebUI** connects to local **Ollama** (GPU-accelerated LXC) as primary, with **OpenRouter** as cloud fallback
- **n8n** automations include Home Assistant webhooks, Immich triggers, and scheduled tasks
- **Watchtower** sends Discord notifications on successful container updates
- **AdGuard Home** serves DNS for the entire LAN — blocks trackers at network level
- **Duplicati** backs up all service data with encryption

## Usage

```bash
# Start all services
docker compose up -d

# View logs for a specific service
docker logs -f n8n

# Update a single service
docker compose pull openwebui && docker compose up -d openwebui
```

Create a `.env` file with your secrets (see variable names in `docker-compose.yml`).
