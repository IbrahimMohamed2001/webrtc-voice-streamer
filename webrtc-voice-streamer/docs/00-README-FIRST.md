# WebRTC Voice Streaming Backend - Handover Documentation

## 🎯 The Elevator Pitch

This is a **Home Assistant Add-on** that provides **real-time, bidirectional voice communication** using WebRTC technology. Think of it as a "voice intercom system" for your smart home - enabling near-zero-latency audio streaming between browsers, mobile devices, and Home Assistant media players.

### What It Actually Does

1. **Voice Sending**: Capture microphone audio from any browser/device and stream it to the server
2. **Voice Receiving**: Play received audio streams through device speakers or Home Assistant media players
3. **MP3 Streaming**: Provide standard HTTP MP3 streams for legacy clients (e.g., VLC, media_player entities)

### Why This Exists

Standard Home Assistant audio integrations have **high latency** (2-10 seconds) due to buffering and protocol overhead. WebRTC provides **sub-second latency** (<500ms) by:

- Using peer-to-peer media paths where possible
- Avoiding unnecessary buffering
- Leveraging browser-native audio codecs (Opus)

## 📦 What You're Looking At

```
webrtc_backend/
├── README.md                                      # Root repository README
├── repository.yaml                                # HA Add-on repository configuration
└── webrtc-voice-streamer                          # The actual Add-on directory
    ├── audio_stream_server.py                     # MP3 HTTP streaming server
    ├── build.yaml                                 # Build configuration for Home Assistant
    ├── config.yaml                                # Home Assistant Add-on manifest
    ├── Dockerfile                                 # Multi-stage build (Node + Python)
    ├── docs                                       # Handover and developer documentation
    │   ├── 00-README-FIRST.md                     # ← This file (Start here)
    │   ├── 01-SETUP-GUIDE.md                      # Installation and configuration guide
    │   ├── 02-ARCHITECTURE.md                     # Deep dive into system design
    │   ├── 03-DECISION-LOG.md                     # Why things are the way they are
    │   ├── 04-GOTCHAS.md                          # Known issues and workarounds
    │   ├── 05-BUILD-RELEASE.md                    # Build & release procedures
    │   ├── architecture-map.md                    # Visual architecture map
    │   ├── HANDOVER-SUMMARY.md                    # Project status handover summary
    │   ├── ONBOARDING-CHECKLIST.md                # New developer checklist
    │   └── TESTING.md                             # Testing strategies and scripts
    ├── frontend                                   # TypeScript/Lit Lovelace cards
    │   ├── package.json                           # NPM dependencies
    │   ├── package-lock.json                      # NPM lockfile
    │   ├── rollup.config.js                       # Rollup bundler configuration
    │   ├── src                                    # Source code for Lovelace cards
    │   │   ├── styles.ts                          # Shared CSS styles
    │   │   ├── types.ts                           # TypeScript interfaces and types
    │   │   ├── voice-receiving-card.ts            # Card to receive audio from HA
    │   │   ├── voice-receiving-editor.ts          # Editor for receiving card
    │   │   ├── voice-sending-card.ts              # Card to send audio to HA
    │   │   ├── voice-sending-editor.ts            # Editor for sending card
    │   │   ├── voice-streaming-card-dashboard.ts  # Dashboard wrapper
    │   │   └── webrtc-manager.ts                  # Core WebRTC logic
    │   ├── tsconfig.json                          # TypeScript build configuration
    │   └── tsconfig.typecheck.json                # TypeScript strict type checking
    ├── icon.png                                   # Add-on icon
    ├── license_middleware.py                      # License validation middleware
    ├── logo.png                                   # Add-on logo
    ├── register_frontend.py                       # Registers Lovelace cards with HA
    ├── requirements.txt                           # Python dependencies
    ├── run.sh                                     # Add-on entrypoint script
    ├── ssl-setup.sh                               # Autonomous SSL certificate management
    ├── tests                                      # Test scripts
    │   └── verify_autossl.sh                      # SSL verification script
    └── webrtc_server_relay.py                     # Main signaling server (WebRTC + WebSocket)
```

## 🚀 Quick Start (For Developers)

### Prerequisites

- Home Assistant OS or Supervised installation
- Docker (for local development)
- Node.js 20+ (for frontend development)
- Python 3.10+ (for backend development)

### 5-Minute Setup

```bash
# 1. Clone the repository
git clone https://github.com/KarimTIS/webrtc-voice-streamer.git
cd webrtc-voice-streaming

# 2. Install frontend dependencies
cd frontend
npm install

# 3. Build frontend
npm run build

# 4. Start backend (development)
cd ..
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python webrtc_server_relay.py
```

### Testing Locally

```bash
# Terminal 1: Start backend
python webrtc_server_relay.py

# Terminal 2: Start frontend dev server
cd frontend
npm run dev

# Browser: Open http://localhost:8080
# You should see the CA certificate download page
```

## 📚 Documentation Navigation

| Document                    | Purpose                        | When to Read            |
| --------------------------- | ------------------------------ | ----------------------- |
| **01-SETUP-GUIDE.md**       | Installation and configuration | First-time setup        |
| **02-ARCHITECTURE.md**      | System design and data flow    | Understanding internals |
| **03-DECISION-LOG.md**      | Architectural decisions        | Making changes          |
| **04-GOTCHAS.md**           | Known issues and workarounds   | Troubleshooting         |
| **ONBOARDING-CHECKLIST.md** | New developer tasks            | Getting started         |

## 🔑 Key Concepts

### WebRTC Signaling

The server doesn't relay media - it facilitates **peer-to-peer connections** between clients. The "signaling" process exchanges SDP (Session Description Protocol) offers/answers via WebSocket to establish direct media paths.

### MediaRelay Pattern

When a sender connects, the server subscribes to their media track using `aiortc.contrib.media.MediaRelay`. This allows **multiple receivers** to subscribe to the same track without the sender needing to know about them.

### Host Networking

The add-on uses `host_network: true` to bypass Docker's NAT. This is **required** for WebRTC because:

- ICE candidates need real IP addresses
- UDP hole punching works better without NAT
- Lower latency (no network translation overhead)

### SSL Autonomy

The system automatically handles HTTPS certificates through a **cascade strategy**:

1. Use existing Home Assistant certificates (if available)
2. Use Ingress mode (if behind HA proxy)
3. Generate self-signed CA (fallback for LAN access)

## 🎯 Use Cases

### 1. Two-Way Intercom

- Place a tablet at your front door with `voice-receiving-card`
- Add `voice-sending-card` to your phone dashboard
- Click to talk/listen in real-time

### 2. Baby Monitor

- Old phone in baby's room with `voice-sending-card` (auto_start: true)
- Your phone with `voice-receiving-card` (auto_play: true)
- Instant audio when baby cries

### 3. Media Player Announcements

- Send microphone audio to Home Assistant media players via MP3 stream
- Broadcast announcements to whole house
- No need for separate microphone server

## 📊 Current Status

| Metric            | Value                               |
| ----------------- | ----------------------------------- |
| **Version**       | 1.1.6 (backend), 1.2.0 (frontend)   |
| **Latency**       | <500ms (typical LAN)                |
| **Codec**         | Opus (WebRTC), MP3 (HTTP streaming) |
| **Max Receivers** | Unlimited (MediaRelay fan-out)      |
| **SSL**           | Automatic (self-signed or HA certs) |
| **Network**       | Host mode (required)                |

## 🆘 Immediate Help

**Problem**: Can't connect to server  
**Solution**: Check `/config/www/voice_streaming_backend/server_state.json` for active port

**Problem**: No audio playing  
**Solution**: Browser requires user interaction - click anywhere on page first

**Problem**: Certificate warnings  
**Solution**: Download CA from `https://<IP>:8443/ca.crt` and install on device

**Problem**: Port conflict  
**Solution**: Server auto-hunts for available port - check logs for actual port

---

**Next Step**: Read [01-SETUP-GUIDE.md](./01-SETUP-GUIDE.md) for installation instructions.
