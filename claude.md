# ark-sa-server

## Overview

This repository provides a Docker container for running an ARK Survival Ascended (ARK SA) dedicated game server on Linux. It's a fork of https://github.com/Johnny-Knighten/ark-sa-server created to fix some issues.

## What It Does

The container wraps the Windows-based ARK SA server and runs it on Linux using Wine/Proton (specifically GloriousEggroll's wine-ge-custom build). It provides:

- Automated server installation, updates, and management
- Configuration via environment variables and INI files
- Scheduled server restarts, updates, and backups via cron
- Automatic mod deployment and management
- Server clustering support
- RCON support for remote administration

## Key Components

### Docker Container Structure

- **Base Image**: `steamcmd/steamcmd:ubuntu-22`
- **Proton/Wine**: Uses GE-Proton8-21 to run the Windows server on Linux
- **Supervisor**: Manages multiple processes (server, cron, etc.)
- **Volumes**:
  - `/ark-server/server` - Server and mod files
  - `/ark-server/logs` - Log files
  - `/ark-server/backups` - Automated backups
  - `/ark-server/cluster` - Cluster data (when clustering is enabled)

### Scripts (in `bin/`)

- `system-bootstrap.sh` - Container entrypoint, initializes the environment
- `ark-sa-bootstrap.sh` - Sets up the ARK server on first run
- `ark-sa-server.sh` - Launches the ARK server with configured parameters
- `ark-sa-updater.sh` - Updates the server files via SteamCMD
- `ark-sa-backup.sh` - Creates backups of server save data

### Configuration System

**config_from_env_vars/** - Python script that generates ARK INI configuration files from environment variables using the pattern:
```
CONFIG_<file>_<section>_<variable>=<value>
```

This allows users to configure game settings (XP rates, gathering rates, player stats, etc.) via Docker environment variables without manually editing INI files.

### Main Environment Variables

- Server settings: `SERVER_NAME`, `SERVER_PASSWORD`, `ADMIN_PASSWORD`, `MAX_PLAYERS`
- Network: `GAME_PORT`, `QUERY_PORT`, `RCON_PORT`, `EPIC_PUBLIC_IP`
- Map: `MAP` (default: TheIsland_WP)
- Updates: `UPDATE_ON_BOOT`, `SCHEDULED_UPDATE`, `UPDATE_CRON`
- Restarts: `SCHEDULED_RESTART`, `RESTART_CRON`
- Backups: `BACKUP_ON_STOP`, `SCHEDULED_BACKUP`, `BACKUP_CRON`, `RETAIN_BACKUPS`
- Mods: `MOD_LIST` (comma-separated mod IDs)
- Clustering: `CLUSTER_ID`, `CLUSTER_DIR`, `NO_TRANSFER_FROM_FILTERING`
- Config management: `MANUAL_CONFIG` (bypass automatic config generation)

## Architecture

1. Container starts via `system-bootstrap.sh`
2. Bootstrap script installs/updates the ARK server via SteamCMD
3. Config files are generated from environment variables (unless `MANUAL_CONFIG=True`)
4. Mods are downloaded and installed if specified
5. Supervisor launches:
   - Cron daemon (for scheduled tasks)
   - ARK server process via `ark-sa-server.sh`
6. Server runs using Wine/Proton compatibility layer

## Features

- **Automated Updates**: Can update on boot or on a schedule
- **Automated Backups**: Backup on stop, scheduled, or before updates/restarts
- **Mod Support**: Automatic downloading and updating of CurseForge mods
- **Clustering**: Multiple servers can share transfer data for cross-server character/item transfers
- **RCON**: Remote console access for administration
- **Flexible Config**: Choose between environment variable-based config or manual INI file management

## Current Version

This fork is tracking the **`next`** branch at version **2.2.2-next.2** (commit d55085a)

Upstream main branch: 2.2.1

## Recent Changes

**Next branch (2.2.2-next.2)** - Unreleased pre-release fixes:
- Fixed config error and missing xaudio2 DLL dependency for version 77.46 (#39)
- Added initial steamcmd call to prevent "Missing Config" error (#39)
- Fixed steamcmd args - moved `force_install_dir` before `login` (#38)

**Released versions:**
- v2.2.1: Fixed leading directory components in backup archives (#36)
- v2.2.0: Enabled config variables with array index syntax like `VariableName[index]` (#34)
- v2.1.0: Server clustering support

## Why This Fork?

This is a fork of https://github.com/Johnny-Knighten/ark-sa-server tracking the `next` branch to get the latest bug fixes before they're released to main.
