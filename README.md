# PocketLAN for Fedora Linux

PocketLAN shares one selected folder with devices on a trusted local network.
The Linux desktop application is distributed as a self-contained AppImage; it
includes Electron, the PocketLAN server, and the browser client.

## What you need

- Fedora Linux (or a compatible 64-bit Linux distribution)
- Node.js 22 or later and npm, only when developing or building the AppImage

Install the build prerequisites on Fedora:

```bash
sudo dnf install nodejs npm
```

## One-command dependency install

This repository uses npm workspaces. From the repository root, install every
dependency for the client, server, and desktop GUI with one command:

```bash
npm install
```

Do not run `npm install` inside `client`, `server`, or `pocketlan-gui`.

## Build the AppImage

From the repository root:

```bash
npm run build:appimage
```

The resulting portable application is written here:

```text
pocketlan-gui/release/PocketLAN-1.0.2.AppImage
```

Make it executable and start it:

```bash
chmod +x pocketlan-gui/release/PocketLAN-1.0.2.AppImage
./pocketlan-gui/release/PocketLAN-1.0.2.AppImage
```

Select a folder, press **Start server**, and copy a displayed LAN URL to a
device on the same trusted network.

The AppImage uses a writable per-user data directory for temporary uploads and
settings. It never writes temporary files into the read-only AppImage mount.

## Fix for the former AppImage crash

Versions before `1.0.2` could crash after a folder was selected because they
attempted to create an upload directory inside the read-only AppImage mount.
This is fixed in `PocketLAN-1.0.2.AppImage`: both the launcher and embedded
server now select a writable per-user directory.

Make sure you are launching the new `1.0.2` file, not a copied older AppImage.
If your system has a restricted home-directory setup, start
the AppImage from a terminal with an explicitly writable upload location:

```bash
mkdir -p "$HOME/.local/share/pocketlan/uploads"
POCKETLAN_TEMP_UPLOAD_DIR="$HOME/.local/share/pocketlan/uploads" \
  ./pocketlan-gui/release/PocketLAN-1.0.2.AppImage
```

If Fedora cannot mount AppImages because FUSE support is unavailable, use:

```bash
./pocketlan-gui/release/PocketLAN-1.0.2.AppImage --appimage-extract-and-run
```

## Development

Run the browser client and server together:

```bash
npm run dev
```

- Browser UI: `http://localhost:5173`
- Backend API: `http://localhost:4242`

Run the Electron desktop GUI in development:

```bash
npm run dev:gui
```

Useful individual commands:

```bash
npm run dev:server
npm run dev:client
npm run build
npm run build:gui
```

## Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `PORT` | `4242` | Backend port for browser/server development |
| `HOST` | `0.0.0.0` | Backend bind address |
| `SHARED_ROOT` | `./SharedFiles` | Folder exposed by the browser/server workflow |
| `APP_PIN` | empty | Optional PIN/password lock |
| `MAX_UPLOAD_SIZE_MB` | `4096` | Per-file upload limit |
| `POCKETLAN_TEMP_UPLOAD_DIR` | per-user app data | Writable temporary-upload folder used by the AppImage |

Example browser/server development run with a chosen folder:

```bash
SHARED_ROOT="$HOME/Videos" APP_PIN="1234" npm run dev
```

## Firewall

Only on trusted private networks, allow the desktop launcher’s configured port
(default `3000`) if other devices cannot connect:

```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

For browser/server development, allow ports `4242` and `5173` instead.

## Security

PocketLAN is for trusted local networks only. Do not expose it to the public
internet or forward its port from a router. Enable a PIN when other people may
be present on the same network.
