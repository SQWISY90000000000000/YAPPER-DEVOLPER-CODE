# 01 — Installer

## Original file

| Property | Value |
|---|---|
| File name | `yapper-windows-x64.exe` |
| Size | 96,719,248 bytes (92.24 MiB) |
| Format | NSIS (Nullsoft Scriptable Install System) |
| Compression | LZMA2 / LZMA / BCJ2, solid |
| Product name | Yapper |
| Product version | 0.1.1-prealpha |
| Company | Yapper |
| Copyright | Copyright © 2026 Yapper |
| Code signing | **Not signed** |
| SHA-256 | `AA31FCFFB8920E6E08B916D191B2E72D175AD9FF826A43499417E60C04EBEE19` |

The file `yapper-windows-x64.exe` in this folder is a **hard link** to the original installer at the workspace root. It is the same file data and does not consume an additional 92 MiB.

## What kind of package it is

The installer is an NSIS wrapper around a packaged Electron application. It is not an installer for a server or for source code. Its payload is the complete Windows desktop runtime:

- `Yapper.exe` — Electron main executable
- `resources/app.asar` — application code and assets
- `resources/app.asar.unpacked` — native Node modules loaded outside the archive
- Chromium/Electron runtime files: `*.dll`, `*.pak`, `*.bin`, `locales/`, `icudtl.dat`
- Electron and Chromium license files
- `resources/elevate.exe` — elevation helper

## Extracted payload

The installer was unpacked into `02-application/` with 7-Zip:

```powershell
7za.exe x yapper-windows-x64.exe -o02-application -y
```

Result: **114 files**, **26 directories**, **330,000,844 bytes** uncompressed. The extracted `02-application/` folder adds a `README.md` (115 files).

> **Known issue:** the installed application opens a window, but its bundled central web UI throws `Missing required env: APP_CENTRAL_API_BASE` and renders a blank content area. This affects the payload, not the installer. See [`../04-documentation/07-run-and-modify.md`](../04-documentation/07-run-and-modify.md) for the tested patch and repack procedure.

### Main payload files

| File | Size (bytes) | Purpose |
|---|---:|---|
| `Yapper.exe` | 201,377,792 | Electron main executable |
| `resources/app.asar` | 30,033,372 | Packaged application code and assets |
| `resources.pak` | 5,926,839 | Chromium resources |
| `icudtl.dat` | 10,466,624 | Chromium ICU data |
| `chrome_100_percent.pak` | 147,743 | Chromium UI resources (100% scale) |
| `chrome_200_percent.pak` | 220,631 | Chromium UI resources (200% scale) |
| `snapshot_blob.bin` | 331,874 | V8 startup snapshot |
| `v8_context_snapshot.bin` | 702,157 | V8 context snapshot |
| `d3dcompiler_47.dll` | 4,916,728 | Direct3D shader compiler |
| `ffmpeg.dll` | 3,001,344 | Media codecs |
| `libEGL.dll` | 493,056 | EGL graphics |
| `libGLESv2.dll` | 8,019,968 | OpenGL ES |
| `vk_swiftshader.dll` | 5,577,728 | Software Vulkan renderer |
| `vulkan-1.dll` | 903,680 | Vulkan loader |
| `resources/elevate.exe` | 107,520 | Elevation helper |
| `LICENSE.electron.txt` | 1,096 | Electron license |
| `LICENSES.chromium.html` | 12,278,238 | Chromium third-party licenses |
| `locales/*.pak` | 55 files | Chromium translations |

### Native modules kept outside `app.asar`

`resources/app.asar.unpacked/node_modules/` contains:

- `keytar` — Windows Credential Manager integration (`.node`)
- `uiohook-napi` — global keyboard/mouse hook (`.node`)

The `uiohook-napi` package includes prebuilt binaries for several platforms:

```
darwin-arm64, darwin-x64, linux-arm64, linux-loong64,
linux-x64, win32-arm64, win32-x64
```

Only the `win32-x64` binary is used by this Windows build, but the package contains the others.

## Next step

The application archive was extracted separately into `03-source/`:

```powershell
npx @electron/asar extract 02-application/resources/app.asar 03-source
```

See [`../02-application/README.md`](../02-application/README.md) and [`../03-source/README.md`](../03-source/README.md).
