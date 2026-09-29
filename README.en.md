# ComicFolk Sync Protocol Server (cfsp-server)

*English | [日本語](README.md)*

A self-hosted server implementation for [ComicFolk](https://github.com/toshi200x). It handles RAR/ZIP/self-scanned PDF archive parsing, page extraction, thumbnail generation, and syncing of reading history / tags / bookmarks on the server side, so the client (the ComicFolk app) doesn't need any archive-parsing logic of its own and can just talk to the server.

- Nothing to install besides Java (the SQLite driver and everything else is bundled)
- Works the same way on Windows / Linux / macOS
- Designed for self-hosting by a single user (one server = one user)
- Update to a new version with one click from the admin panel (v0.1.27 and later)
- Per-book margin crop and image correction settings are saved on the server and shared across your devices (v0.1.29 and later, with ComicFolk app v1.23 or later)

This repository only distributes **pre-built binaries**. The source code is not public.

## Download

Grab the latest `cfsp-server-<version>.zip` from [Releases](../../releases). (The `.zip.sig` file next to it is a digital signature; you don't need it when installing manually.)

## Requirements

- **Java 17 or later** (either a JRE or a JDK works). Using an older Java results in a confusing `UnsupportedClassVersionError`, so make sure you're on 17+.
- (Optional) **ffmpeg** (5.0 or later recommended) — only needed for generating thumbnails from AVIF-format archives. Everything else (including JPEG/PNG thumbnails) works fine without it.

## Running it

Unzip and run the launcher script in the `bin` folder.

```
# Linux/Mac
./bin/cfsp-server

# Windows
bin\cfsp-server.bat
```

This starts the admin panel (by default at `http://<this machine's address>:7878/`).

- Updates and restarts from the admin panel are carried out by this launcher script. **Always start the server with the launcher in `bin`** (if you start it directly with `java -jar` or similar, updating and restarting from the admin panel won't be available).
- You can pass options to Java with the `JAVA_OPTS` environment variable (e.g. `JAVA_OPTS=-Xmx4g`). The launcher uses the Java in `JAVA_HOME` if it's set, otherwise `java` on your `PATH`.

## First-time setup

1. When you open the admin panel in a browser, you'll first be asked to **set a password for the admin panel** (8+ characters).
2. After setting the password and logging in, specify one or more **library sources** (folders containing your archives) on the dashboard. Either browse to them with the folder picker or type the path directly, then save.
3. Saving automatically issues a connection token and starts the library API (port 8787 by default) right away, with no server restart needed.
4. Scan the QR code shown on the dashboard from the "Add server" screen in the ComicFolk app to register the address and token in one go.

## Updating

From v0.1.27, you can update to a new version from "Server updates" in the admin panel.

1. Click "Check for updates" to look up the latest release in this repository. If there's a newer version, what's new in it is shown.
2. Click "Update and restart" to download, install and restart automatically (usually 1–2 minutes; the app can't connect in the meantime). When it's done, the admin panel reloads by itself, and after you log in again it shows the result.

- By default, the server checks once a day whether a new version is available (you can turn this off with the checkbox in the admin panel). It only checks; it never installs anything on its own.
- When a new version is available, it's also shown under "Notifications" in the ComicFolk app's settings screen (with a badge on the settings icon; requires an app version that supports notifications).
- Release zips are digitally signed by the developer, and the server verifies the signature before installing. A file whose signature doesn't match (corrupted, or not an official release) is never installed.
- If the updated version crashes right after starting, the server is automatically rolled back to the previous version and restarted, and the admin panel tells you the update failed. The previous version is kept in the `lib.prev` folder of the install location.
- With "Update from a file", you can also upload a release zip (and its `.zip.sig`) that you downloaded yourself (for servers that can't reach GitHub).

### Moving from v0.1.11 or earlier (one time, manually)

Launcher scripts from v0.1.11 and earlier don't support updating from the admin panel, so replace the install manually once to get to v0.1.27.

1. Stop the server.
2. Unzip v0.1.27 and from now on start the server with the launcher in that folder's `bin` (if you registered the server with Task Scheduler or similar, point it at the new location).
3. Your settings, index and thumbnails live in the data directory described below, so they carry over as-is.

From then on you can update from the admin panel. The install folder stays where it is (only its contents are replaced), so you don't need to touch your Task Scheduler or similar setup again.

- If a future version needs a change to the launcher script itself, the admin panel will say so, and only that version needs to be replaced manually the same way.

## Where data is stored

The config file, index DB and thumbnails are **not** stored in the current working directory at runtime; by default they go into a fixed folder under your home directory.

```
~/.comicfolk-sync-server/
  cfsp-admin.properties   # admin password, library sources, connection token, etc.
  cfsp-index.db           # persistent index (SQLite; also holds history, tags, bookmarks, etc.)
  thumbnails/             # generated thumbnails
  update/                 # working files for updates from the admin panel
```

So replacing the install location (the folder containing `bin/` and `lib/`) keeps all of the above.

You can override these locations with environment variables.

| Variable | What it controls | Default |
|---|---|---|
| `CFSP_DATA_DIR` | The data directory itself | `~/.comicfolk-sync-server` |
| `CFSP_ADMIN_CONFIG_PATH` | Overrides just the config file path | `$CFSP_DATA_DIR/cfsp-admin.properties` |
| `CFSP_DB_PATH` | Overrides just the index DB path (thumbnails are stored in the same directory as this file) | `$CFSP_DATA_DIR/cfsp-index.db` |
| `CFSP_ADMIN_PORT` | Admin panel port | `7878` |
| `CFSP_RESCAN_INTERVAL_SEC` | Interval for periodic library rescans (seconds) | `300` |

If you run the server as the "SYSTEM" account from Windows Task Scheduler, the home directory resolves to SYSTEM's, so it's a good idea to set `CFSP_DATA_DIR` explicitly.

## Ports

| Port | Purpose |
|---|---|
| 7878 | Admin panel (web UI). Configurable via `CFSP_ADMIN_PORT` |
| 8787 | Library API (what the ComicFolk app connects to). Currently there's no UI on the dashboard to change it. To change it, edit `api.port` in `cfsp-admin.properties` directly, or start the server with the `CFSP_PORT` environment variable |

Both are intended to be used within your LAN or over a VPN such as Tailscale; exposing them directly to the internet is not supported.

## Running as a service (always-on)

There's no Windows service, systemd unit, or Docker image yet. If you need it to stay running long-term, set up your OS's standard mechanism (Task Scheduler on Windows, a systemd unit on Linux, etc.) to start the launcher script in `bin`.

- Updates and "Restart now" from the admin panel work without relying on your service manager: the launcher script restarts the server by itself.
- If the server exits for any other reason, the launcher script exits too (set up automatic restarts on crashes in your service manager).

## Speech-balloon translation requirements

The ComicFolk app's "automatic speech-balloon translation" (app v1.22 or later) has this server translate the dialogue of books (using the Google Gemini API; the API key is set in the app's Settings and passed to the server encrypted). Text-region detection uses ONNX Runtime, so it only runs on certain OS/CPU combinations. **On unsupported systems only translation is disabled; everything else keeps working** (the reason is shown in the admin panel).

| OS | CPU | Extra requirements |
|---|---|---|
| Windows | x64 | None (the Visual C++ runtime is bundled and fills in whatever the PC is missing). If it still can't load, follow the admin panel's guidance and install the [Microsoft Visual C++ Redistributable (x64)](https://aka.ms/vs/17/release/vc_redist.x64.exe) |
| Linux | x64 / ARM64 | None (glibc 2.27 or later: Ubuntu 18.04+, Debian 10+, RHEL 8+, etc.) |
| macOS | Apple Silicon | None |

Translation isn't available on Intel Macs, musl-based systems such as Alpine Linux, 32-bit OSes, or Windows on ARM.

## Known limitations

- No Docker distribution or bundled Tailscale sidecar yet
- Authentication uses a single Bearer token; per-device token issuance/revocation isn't implemented
- The thumbnail sync API doesn't support incremental sync (full fetch only)

## License / support

This is published as a byproduct of personal use. Since the source code is not public, questions via Issues will be answered on a best-effort basis, with no guarantee of support.
