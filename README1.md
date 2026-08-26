<div align="center">

# AIpolloSync: Remote P2P Media Server & Streaming Skill for Hermes

[English](README.md) | [简体中文](README_ZH.md) | [日本語](README_JA.md) | [Deutsch](README_DE.md) | [Español](README_ES.md)

</div>

---

## 📖 Overview & Core Value

**AIpolloSync** is a personal media file remote sharing service. It starts a local Flask + WebRTC media server, establishes an outbound FRP tunnel for public internet access, and exposes your local video files through a WhatsApp-integrated AI agent interface.

**Core value**: Access your remote media file library anytime, anywhere via WhatsApp connected to Hermes. Media playlists support AIpollo player playback.

---

## ⚙️ Prerequisites

- Local Hermes environment successfully deployed
- Trust/add exception for `frpc.exe` in your security software if blocked

---

## 🚀 Step-by-Step Installation & Usage

1. **Install**: Download and install the `AIpolloSync` skill via HermesHub.
2. **Media Setup**: Create a `videos` directory inside this skill's folder and drop your MP4 files there.
3. **Integration**: Link and configure your WhatsApp channel in Hermes.
4. **Launch**: Start the `AIpolloSync` skill in Hermes.
5. **Interact (LLM-Driven)**: Chat with the AI naturally in your channel. For example:
   - *"Show me my video list."*
   - *"Do I have any movie to watch?"*
   - *"Play the video about cat."*
6. **Play**: Click the generated link from the response to play your video.

---

## 🔒 Security & Network Disclosure

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

---

## 🛡️ Best Practice Recommendations

* **Dedicated Server / Virtual Machine**: For optimal security, we strongly recommend running this media server skill on a standalone secondary device or within an isolated Virtual Machine (VM) rather than on your primary workstation.
* **Routine Maintenance**: Keep your host operating system, Hermes environment, and security patches updated regularly.

---

## 💻 Platform Compatibility

* **Current Support**: Windows (x64)
* **Roadmap**: Linux / macOS support is under active development.

*If you need support for other platforms or encounter network traversal issues, please open a GitHub Issue or reach out to us. Thank you for your support and trust!*