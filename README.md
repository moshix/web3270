# 🖥️ web3270 — Browser-Based 3270 Terminal Emulator

> **A modern, full-featured TN3270 terminal emulator that runs entirely in your browser. Single binary, zero dependencies.**

---

## 📦 Installation

### Download

Pre-built binaries are available for all major platforms:

| Platform         | Binary Name                          |
|------------------|--------------------------------------|
| Linux (AMD64)    | `web3270-x.x.x.x-linux-amd64`       |
| Linux (ARM64)    | `web3270-x.x.x.x-linux-arm64`       |
| macOS (Intel)    | `web3270-x.x.x.x-darwin-amd64`      |
| macOS (Apple Silicon) | `web3270-x.x.x.x-darwin-arm64` |
| Windows (AMD64)  | `web3270-x.x.x.x-windows-amd64.exe` |


### Quick Start

```bash
# Start web3270 on port 8080, pointing at a mainframe host
./web3270 -listen :8080 -host mainframe.example.com -port 3270
```

Then open your browser to `http://localhost:8080` — no plugins, no Java applets, no native installs.

---

## ⚙️ Command-Line Options

```
Usage: web3270 [-listen :port] [-host hostname] [-port tn3270port]
               [-model 3279-N] [-lock] [-tls-cert file -tls-key file]
```

| Flag           | Description                                      | Default          |
|----------------|--------------------------------------------------|------------------|
| `-listen`      | HTTP/HTTPS listen address                        | `:8080`          |
| `-host`        | Default TN3270 host shown in the connection form | `localhost`      |
| `-port`        | Default TN3270 port shown in the connection form | `3270`           |
| `-model`       | Terminal model: `3279-2`, `3279-3`, or `3279-4`  | `3279-4`         |
| `-lock`        | Lock host/port — users cannot change the target  | off              |
| `-tls-cert`    | TLS certificate file (enables HTTPS)             | —                |
| `-tls-key`     | TLS private key file (requires `-tls-cert`)      | —                |

### Examples

```bash
# Basic — connect to a local MVS system
./web3270 -listen :8080 -host localhost -port 3270

# HTTPS with locked host (kiosk mode)
./web3270 -listen :443 -host mvs.example.com -port 2300 -lock \
          -tls-cert cert.pem -tls-key key.pem

# Use a smaller terminal model
./web3270 -listen :9090 -host zos.internal -port 3270 -model 3279-2
```

### Kiosk Mode (`-lock`)

When `-lock` is specified, the host and port fields in the browser are disabled and the client auto-connects on page load. Ideal for shared terminals, lab environments, or public-facing deployments where users should only connect to one specific host.

---

## 🔌 Connecting

| Field        | Description                              | Default  |
|--------------|------------------------------------------|----------|
| **Host**     | Hostname or IP of the TN3270 server      | *(from `-host` flag)* |
| **Port**     | TN3270 port                              | *(from `-port` flag)* |
| **Model**    | Terminal model (see below)               | Model 4  |
| **Codepage** | EBCDIC codepage for character mapping    | CP 1047  |

### 🖵 Terminal Models

| Model  | Dimensions| Use Case               |
|--------|-----------|------------------------|
| 2      | 24 × 80   | Standard 3270          |
| 3      | 32 × 80   | Extended rows          |
| 4      | 43 × 80   | Large screen (default) |
| 5      | 27 × 132  | Wide format            |
| custom |           | up to 90x250           |

### 🌍 Supported Codepages

> CP 1047 (default), CP 037 (US), CP 500 (International), CP 273 (German),
> CP 275 (Brazil), CP 277 (Danish), CP 278 (Swedish), CP 280 (Italian),
> CP 284 (Spanish), CP 285 (UK), CP 297 (French), CP 310 (APL/Graphics),
> and "bracket" (special).

### URL Parameters

You can bookmark or share connections using URL query parameters:

```
http://your-server:8080/web3270?host=mvs.example.com&port=3270&model=4&codepage=1047
```

Parameters: `host`, `port`, `model`, `codepage`. These override the command-line defaults.

---

## 🚀 Features

### 📂 File Transfer (IND$FILE)

web3270 supports **bidirectional file transfer** with mainframe hosts using the IND$FILE protocol.

#### Two Transfer Protocols

| Protocol | Description                         | Best For               |
|----------|-------------------------------------|------------------------|
| **DFT**  | Distributed File Transfer (modern)  | Most systems           |
| **CUT**  | Command for Upload/Transfer (legacy)| Older hosts            |

#### Download (Receive)

Retrieve a file from the host to your local machine:

1. Open the **File Transfer** dialog
2. Select **Receive**
3. Enter the host dataset or file name
4. Choose **ASCII** (text) or **Binary** mode
5. Click **Transfer** — the file downloads through your browser

#### Upload (Send)

Send a local file to the host:

1. Open the **File Transfer** dialog
2. Select **Send**
3. Choose your local file
4. Select the **host type**: TSO, VM/CMS, or CICS
5. Configure allocation options:

   **TSO options:**
   - Record Format: Fixed (F), Variable (V), Undefined (U)
   - LRECL (Logical Record Length)
   - BLKSIZE (Block Size)
   - SPACE allocation (Primary/Secondary, Tracks/Cylinders/Avblock)
   - Append mode

   **VM/CMS options:**
   - Record Format: Fixed (F), Variable (V)
   - LRECL

   **CICS:**
   - Basic binary transfer

6. Click **Transfer**

#### Data Modes

- **ASCII (Text)** — Automatic EBCDIC ↔ ASCII conversion with CR/LF handling
- **Binary** — Raw byte transfer, no conversion

#### DFT Buffer Size

Configurable from 256 to 32,768 bytes (default 4,096). Increase for faster transfers on reliable links; decrease for slow or unreliable connections.

---

### ⚡ Delta Screen Updates (Performance)

web3270 uses an intelligent **delta rendering** system to minimize bandwidth and maximize responsiveness.

| Update Type     | When Used                            | Data Sent               |
|-----------------|--------------------------------------|-------------------------|
| **Full screen** | First frame, resize, Erase/Write     | All `rows × cols` cells |
| **Delta**       | Subsequent updates (<50% changed)    | Only changed cells      |
| **Fallback**    | >50% of cells changed                | Full screen             |

**Benefits:**
- 🟢 **50–90% bandwidth reduction** on typical screens
- 🟢 **Faster rendering** on slow connections
- 🟢 Uses `requestAnimationFrame` to coalesce rapid updates
- 🟢 Non-blocking WebSocket sends (drops stale frames if buffer full)

---

### 🎨 Color Themes

Seven built-in themes — click to switch instantly, no reload needed.

| Theme            | Style                                  |
|------------------|----------------------------------------|
| **3279**         | 🟢 Authentic IBM 3279 colors (default) |
| **Chalk**        | 🩵 Soft pastel tones                   |
| **Earthsong**    | 🟤 Natural, warm palette               |
| **FunForrest**   | 🟠 Warm forest colors                  |
| **Kolorit**      | 🟣 Vibrant and bold                    |
| **Ubuntu**       | 🟡 Dark Ubuntu terminal                |
| **Green Screen** | 🟩 Classic monochrome green            |

Each theme maps all seven 3270 colors (green, blue, red, pink, turquoise, yellow, white) to a coordinated palette. Reverse video, cursors, and field highlights all adapt automatically.

---

### 🔤 Fonts

Eight monospace fonts available:

| Font               | Notes                          |
|--------------------|--------------------------------|
| **IBM Plex Mono**  | Default — closest to 3270 look |
| Courier New        | Universal fallback             |
| Consolas           | Windows-optimized              |
| Liberation Mono    | Linux standard                 |
| Fira Code          | Modern monospace               |
| Source Code Pro    | Adobe professional             |
| JetBrains Mono    | Developer-focused              |
| System Mono       | OS default                     |

**Font sizes:** 10px, 12px, 14px (default), 16px, 18px, 20px, 24px

Changes apply instantly — no reconnection needed.

---

### ⌨️ Keyboard Remapping

Fully customizable keyboard bindings for all 34 mappable 3270 functions.

#### Default Key Map

| Key                | 3270 Function |
|--------------------|---------------|
| `Enter`            | Enter AID     |
| `Escape`           | Clear         |
| `F1`–`F12`         | PF1–PF12      |
| `Shift+F1`–`F12`   | PF13–PF24     |
| `Ctrl+1`           | PA1           |
| `Ctrl+2`           | PA2           |
| `Ctrl+3`           | PA3           |
| `Right Alt` (Mac)  | Enter         |
| `Right Meta` (Mac) | PA1           |

#### Remapping

1. Open **Settings**
2. Click the function you want to remap
3. Press the new key combination
4. The binding is saved automatically

Use **Reset Defaults** to restore the original key map.

---

### 💾 Local Storage (Persistent Settings)

All preferences are saved automatically in your browser's localStorage and restored on next visit.

| Setting            | Storage Key               |
|--------------------|---------------------------|
| Color theme        | `web3270-theme`           |
| Keyboard map       | `web3270-keymap`          |
| Last host          | `web3270-host`            |
| Last port          | `web3270-port`            |
| Terminal model     | `web3270-model`           |
| Codepage           | `web3270-codepage`        |
| Font family        | `web3270-font`            |
| Font size          | `web3270-fontsize`        |
| Auto-reconnect     | `web3270-autoreconnect`   |
| Connection history | `web3270-history`         |

**Connection history** stores your last 10 host:port combinations with associated model, codepage, and theme — available as a dropdown when typing in the host field.

Storage usage is minimal (~10KB typical). No risk of exceeding browser quota.

---

### 🔄 Auto-Reconnect

When enabled in Settings, web3270 automatically reconnects after a dropped connection using exponential backoff (1s → 2s → 4s → … up to 30s max). The status bar shows reconnection progress.

---

### 🔒 Security

- **TLS/HTTPS** — Use `-tls-cert` and `-tls-key` to serve over HTTPS with WebSocket Secure (wss://)
- **Per-IP connection limiting** — Max 5 connections per IP address
- **Total connection cap** — 50 concurrent WebSocket connections
- **Message size limit** — 2MB max WebSocket message
- **Origin checking** — Same-origin policy enforced

---

## 🖱️ Status Bar

The bottom status bar provides at-a-glance information:

```
Connected  Row: 21  Col: 36  OVR  Connected to host (Model 3) 1 min   web3270 v4.x.x.x ⓘ
```

| Element         | Description                                    |
|-----------------|------------------------------------------------|
| Connection      | 🟢 Connected / 🔴 Disconnected                |
| Row / Col       | Current cursor position                        |
| OVR / INS       | Overwrite or Insert mode                       |
| Status message  | Connection info, transfer progress, errors     |
| Version         | web3270 version number                         |
| ⓘ              | Copyright and info                             |

---

## 🧩 Field Navigation

| Key              | Action                        |
|------------------|-------------------------------|
| `Tab`            | Next unprotected field        |
| `Shift+Tab`      | Previous unprotected field    |
| `NewLine`        | Next field on next row        |
| Arrow keys       | Move cursor within fields     |

- Protected fields are skipped automatically
- Numeric fields accept only `0-9`, `.`, `-`
- Hidden fields mask input (passwords)

---

## 📋 Copy & Paste

- **Copy**: Select text with mouse, then `Ctrl+C`
- **Paste**: `Ctrl+V` — text is inserted respecting field boundaries

---
## Some Important Info

- Downloads go to browser's download folder (no filesystem access)

Screenshot[Screenshot%202026-04-09%20at%2006.41.34.png]

---

## 📝 License

web3270 is copyright 2026 by moshix. All rights reserved.
