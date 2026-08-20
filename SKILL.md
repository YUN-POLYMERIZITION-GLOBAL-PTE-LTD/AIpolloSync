---
name: AIpolloSync
version: 1.0.1
description: This tool automatically downloads and executes the third-party 'frpc' binary from GitHub to establish an active outbound reverse tunnel. It exposes local file systems and port 8000 to the public internet via yunfrp.net subdomains. Additionally, it initiates outbound WebSocket/WebRTC signaling and connects to external STUN servers for peer-to-peer media streaming, enabling remote file access.
author: Hermes User
tags:
  - media
  - video
  - streaming
  - webrtc
  - frp
  - p2p
  - remote-access
---

# AIpolloSync

## Overview

AIpolloSync is a personal media file remote sharing service. It starts a local Flask + WebRTC media server, establishes an outbound FRP tunnel for public internet access, and exposes your local video files through a WhatsApp-integrated AI agent interface.

**Core value**: Access your remote media file library anytime, anywhere via WhatsApp connected to Hermes. Media playlists support AIpollo player playback.

## Prerequisites

- Local Hermes environment successfully deployed
- Trust/add exception for `frpc.exe` in your security software if blocked

## Directory Structure

```
AIpolloSync/
├── SKILL.md              # This file — skill definition
├── requirements.txt      # Python dependencies
├── scripts/              # Python source code
│   ├── media_server_flask.py   # Main entry point — Flask + WebRTC server
│   ├── media_frp_util.py       # FRP tunnel management
│   ├── media_file_util.py      # Media file discovery
│   └── uuid_config.py          # UUID-based subdomain config
├── assets/               # Static assets
│   └── templates/        # HTML templates
│       ├── medias.html
│       └── player.html
├── videos/               # Drop your MP4 files here (create manually)
├── readme.md             # Chinese usage guide
└── CHANGELOG.md          # Version history
```

## Specific Steps

1. **Install**: Download and install the `AIpolloSync` skill via HermesHub.
2. **Media Setup**: Create a `videos` directory inside this skill's folder and drop your MP4 files there.
3. **Integration**: Link and configure your WhatsApp channel in Hermes.
4. **Launch**: Start the `AIpolloSync` skill in Hermes by running:
   ```
   cd scripts && python media_server_flask.py
   ```
5. **Interact (LLM-Driven)**: Chat with the AI naturally in your channel. For example:
   - *"Show me my video list."*
   - *"Do I have any movie to watch?"*
   - *"Play the video about cat."*
6. **Play**: Click the generated link from the response to play your video.

## How It Works

### LLM Agent Mode

This skill operates entirely at the **LLM Action/Tool execution level**. The LLM intelligently understands user intents, translates fuzzy queries into structured API parameters, and hits your local Flask backend to fetch data.

The API endpoint `POST /api/list_files` returns clean JSON data (file lists, file URLs). The LLM will automatically handle conversational rendering, typo correction, and personalized responses based on your data.

### Architecture

```
WhatsApp → Hermes (LLM) → POST /api/list_files → Flask Server (port 8000)
                                ↓
                    FRP Tunnel (yunfrp.net)
                                ↓
                    Public Internet Access
```

- **Flask** serves HTTP routes (API, media files, player page)
- **Flask-SocketIO** handles WebSocket signaling (same port as Flask)
- **aiortc** handles WebRTC P2P data channels for media streaming
- **FRP** tunnels all traffic through FRPS for remote access
- **WebRTC P2P** bypasses FRPS for actual media data, saving bandwidth

## Security & Network Disclosure

### Critical: FRP Tunnel Exposes Local Services to Public Internet

This skill **automatically** downloads and runs the **FRP (Fast Reverse Proxy) client (`frpc`)** upon startup. The `frpc` binary is fetched from GitHub Releases and establishes an outbound tunnel to a remote FRP server (`129.213.174.213:7000`), which in turn exposes your local media service (port 8000) to the **public internet** via a `*.yunfrp.net` subdomain.

**This materially expands your attack surface.** Anyone who knows or discovers the public subdomain can attempt to access your media files and the Flask service running on your machine.

### 1. Automatic Tunnel Behavior (No User Opt-in)

- **Automatic on Startup**: The FRP tunnel starts automatically when `scripts/media_server_flask.py` runs. There is no prompt, no confirmation, and no environment-variable gate.
- **Binary Download**: On first run, `frpc.exe` is downloaded silently from GitHub (`fatedier/frp` releases). Internet access is required.
- **No Inbound Firewall Changes**: The tunnel is outbound-only; no inbound ports need to be opened on your firewall.

### 2. Supply-Chain Risk: Downloaded Binary Execution

- The skill downloads and executes a native binary (`frpc.exe`) from GitHub Releases. Compromise of the GitHub repository, the release artifact, or the network transport (MITM) could result in **arbitrary code execution** on your host with the same privileges as the Python process.
- **Pinned SHA256 Verification**: The code includes hardcoded SHA256 checksums for both the zip archive and the extracted `frpc.exe` binary (version `0.65.0`). The download is rejected if either checksum does not match. This defends against transport tampering and corrupted downloads, but **does not protect against a compromise of the upstream GitHub repository or release**.
- **Version-Locked**: The FRP version is pinned at `0.65.0`. Upgrading requires a code change and SHA256 re-verification. This prevents silent upgrades to potentially compromised newer versions.

### 3. Authentication Status

- **No Authentication Implemented**: The Flask server currently has **no HTTP Basic Auth, no token mechanism, and no access control**. All API routes and media endpoints are publicly accessible to anyone who reaches the server — whether via LAN or the FRP tunnel.
- **Risk**: An unauthenticated third party who discovers the `*.yunfrp.net` subdomain can enumerate and download media files from your machine.

### 4. Remote Server Trust

- The FRP server at `129.213.174.213:7000` is a third-party relay. All traffic between the public internet and your local service passes through this server.
- The FRP tunnel operates in HTTP mode (no TLS termination by FRP server).
- You must trust that this FRP server operator will not inspect, log, or tamper with your traffic.