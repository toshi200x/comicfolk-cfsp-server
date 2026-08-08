# ComicFolk Sync Protocol Server (cfsp-server)

*English | [日本語](README.md)*

A self-hosted server implementation for [ComicFolk](https://github.com/toshi200x). It handles RAR/ZIP archive parsing, page extraction, thumbnail generation, and syncing of reading history / tags / bookmarks on the server side, so the client (the ComicFolk app) doesn't need any archive-parsing logic of its own and can just talk to the server.

- Pure JVM implementation, no extra installation required (the SQLite driver is bundled in the jar)
- Works the same way on Windows / Linux / macOS
- Designed for self-hosting by a single user (one server = one user)

This repository only distributes **pre-built binaries**. The source code is not public.

## Download

Grab the latest `cfsp-server-<version>.zip` from [Releases](../../releases).

## Requirements

- **Java 17 or later** (either a JRE or a JDK works). Using an older Java results in a confusing `UnsupportedClassVersionError`, so make sure you're on 17+.
- (Optional) **ffmpeg** (5.0 or later recommended) — only needed for generating thumbnails from AVIF-format archives. Everything else (including JPEG/PNG thumbnails) works fine without it.

## Running it

Just unzip and run the launcher.

```
# Linux/Mac
./bin/cfsp-server

# Windows
bin\cfsp-server.bat
```

Once started, the admin panel comes up at `http://<this machine's address>:7878/` by default.

## First-time setup

1. Opening the admin panel in a browser first asks you to **set an admin password** (8+ characters).
2. After setting the password and logging in, specify one or more **library sources** (folders containing your archives) on the dashboard. You can browse to them with the folder picker, or type a path directly and save.
3. Saving automatically issues a connection token, and the library API (port 8787 by default) starts immediately without needing a server restart.
4. Scanning the QR code shown on the dashboard from the ComicFolk app's "Add Server" screen registers the address and token together.

## Where data is stored

The config file, index database, and generated thumbnails are stored, by default, in a fixed folder under your home directory — **not** the current working directory the server happens to be launched from.

```
~/.comicfolk-sync-server/
  cfsp-admin.properties   # admin password, library sources, connection token, etc.
  cfsp-index.db           # persistent index (SQLite; also holds history/tags/bookmarks etc.)
  thumbnails/             # generated thumbnail images
```

This means upgrading to a new version is just a matter of swapping out the extracted `bin/`/`lib/` directory — your data carries over regardless of where you extract and run it from.

You can override the storage location via environment variables:

| Environment variable | What it controls | Default |
|---|---|---|
| `CFSP_DATA_DIR` | The data directory itself | `~/.comicfolk-sync-server` |
| `CFSP_ADMIN_CONFIG_PATH` | Just the config file path | `$CFSP_DATA_DIR/cfsp-admin.properties` |
| `CFSP_DB_PATH` | Just the index DB path (thumbnails are stored alongside this file) | `$CFSP_DATA_DIR/cfsp-index.db` |
| `CFSP_ADMIN_PORT` | Admin panel port | `7878` |
| `CFSP_RESCAN_INTERVAL_SEC` | Periodic library rescan interval (seconds) | `300` |

## Ports

| Port | Purpose |
|---|---|
| 7878 | Admin panel (Web UI). Change via `CFSP_ADMIN_PORT` |
| 8787 | Library API (what the ComicFolk app connects to). Fixed at the default for now — there's no dashboard UI to change it yet. To change it, edit `api.port` directly in `cfsp-admin.properties`, or launch with the `CFSP_PORT` environment variable |

Both are intended to be used over a LAN or a VPN such as Tailscale — direct exposure to the public internet is not a supported use case.

## Running as a service

Right now the only supported way to run it is directly via the launcher script — there's no bundled Windows service, systemd unit, or Docker image yet. If you need it to run persistently, set that up yourself using your OS's standard tools (Task Scheduler / NSSM on Windows, a systemd unit on Linux).

## Known limitations

- No Docker distribution or bundled Tailscale sidecar yet
- Authentication is a single Bearer token only — no per-device issuance/revocation yet
- The thumbnail sync API doesn't support incremental sync yet (full list fetch only)

## License / Support

This is published as a byproduct of a personal project. Since the source isn't public, I'll respond to questions via Issues when I can, but I can't promise support.
