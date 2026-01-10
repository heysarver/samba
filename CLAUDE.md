# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Build the Docker image
docker build -t local/samba:dev .

# Build with no cache (for Alpine/Samba version updates)
docker build -t local/samba:dev . --no-cache

# Run container with docker-compose
docker compose up -d

# View logs
docker compose logs -f samba
```

## Architecture

This is a Docker container for Samba file sharing, forked from dperson/samba and optimized for macOS clients.

**Key files:**
- `Dockerfile` - Alpine-based image with smb.conf baked in during build. All global Samba settings are configured here via echo statements.
- `samba.sh` - Entrypoint script that processes environment variables and command-line arguments to dynamically configure shares, users, and options at runtime.
- `docker-compose.yaml` - Local deployment configuration using environment variables for shares/users.

**Configuration flow:**
1. Dockerfile creates base smb.conf with global settings (vfs objects, fruit module, security, performance)
2. At container start, samba.sh reads `SHARE*`, `USER*`, `GLOBAL*`, `GENERIC*` environment variables
3. samba.sh modifies smb.conf and starts smbd (and optionally nmbd)

**Environment variable patterns:**
- `SHARE1`, `SHARE2`, etc. - Define shares with semicolon-separated fields: `name;path;browsable;readonly;guest;users;admins;writelist;comment`
- `USER1`, `USER2`, etc. - Define users: `username;password;uid;group;gid`
- `GLOBAL1`, `GLOBAL2`, etc. - Add global smb.conf options
- `GENERIC1`, `GENERIC2`, etc. - Add section-specific options: `section;parameter = value`

## macOS Optimization

This image includes vfs_fruit module settings optimized for macOS clients. Key settings in Dockerfile:
- `fruit:metadata = stream` - Efficient metadata storage
- `fruit:nfs_aces = no` - Prevents permission issues
- `server signing = disabled` - Critical for performance
- `server multi channel support = yes` - SMB3 multichannel

For macOS clients, create `/etc/nsmb.conf`:
```ini
[default]
signing_required=no
streams=yes
mc_on=yes
mc_prefer_wired=yes
```
