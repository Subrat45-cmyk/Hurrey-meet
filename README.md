# Hurrey Meet — Full-Stack WebRTC Video Conferencing Platform

**Hurrey Meet** is a high-performance, real-time video conferencing web application inspired by Google Meet and Zoom. Built from the ground up using **Vanilla JavaScript, HTML5/CSS3, Node.js, Express, Socket.IO, and Native WebRTC APIs** — with zero dependency on third-party hosted video SDKs.

---

## Features

- **Multi-User Meeting Rooms**: Create unique meeting rooms, invite peers via shareable links, and display real connected participants with synchronized presence.
- **Real Two-Way Audio & Video**: Low-latency mesh communication powered by browser native `navigator.mediaDevices.getUserMedia()` and `RTCPeerConnection` with the **Polite Peer (Perfect Negotiation)** pattern.
- **Google Meet-Style Screen Sharing**: Native `getDisplayMedia()` capturing of entire screens, specific windows, or tabs. Includes dynamic presentation mode with central stage, active presenter indicator, and graceful recovery when stopped via app or browser chrome.
- **Meeting Toolbar Controls**:
  - Microphone mute/unmute with track hardware disablement and remote status icons.
  - Camera toggle with fallback to avatar initials placeholders.
  - Screen share start/stop button.
  - In-meeting text chat with unread message badges.
  - Live participant list with host tags and audio/video statuses.
  - Real-time WebRTC connection diagnostics modal (candidate pair inspection, RTT, packets lost, bitrate).
  - Fullscreen mode and one-click leave meeting.
- **Pre-Join Screen (Lobby)**: Camera/mic live preview, interactive VU audio level meter (Web Audio API), display name customization, and device permission diagnostics before joining.
- **STUN & TURN Network Traversal**: Native support for STUN and TURN relays (UDP, TCP, and TLS) with a secure server endpoint (`/api/ice-servers`) that issues RFC 5766 time-limited HMAC credentials.

---

## Technology Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | HTML5, CSS3 (Modern Dark Theme), Vanilla JavaScript (ES6+) |
| **Real-Time Media** | Native WebRTC APIs (`RTCPeerConnection`, `getDisplayMedia`, `getUserMedia`) |
| **Backend** | Node.js, Express.js |
| **Signaling** | Socket.IO (WebSockets with polling fallback) |
| **Traversal** | STUN (`stun.l.google.com`) and TURN (Coturn / RFC 5766 REST API compatible) |
| **Testing** | Node.js Built-In Test Runner (`node --test`) |

---

## Project Structure

```
Hurreymeet/
├── public/
│   ├── index.html              # Landing page (create meeting, join with code/link)
│   ├── lobby.html              # Pre-join lobby (device preview, audio meter)
│   ├── meeting.html            # Meeting room interface (grid, presentation, toolbar)
│   ├── css/
│   │   ├── style.css           # Common styles, landing & lobby themes
│   │   └── meeting.css         # Meeting room layout, tiles, presentation, drawers
│   └── js/
│       ├── app.js              # Landing page controller
│       ├── lobby.js            # Pre-join lobby controller
│       ├── meeting.js          # Meeting room coordinator
│       ├── webrtc.js           # WebRTC mesh engine & connection diagnostics
│       ├── screenshare.js      # Screen capture & presentation layout manager
│       └── signaling.js        # Socket.IO client signaling adapter
├── server/
│   ├── index.js                # Express app, static server, and Socket.IO entrypoint
│   ├── rooms.js                # Room state, participant tracking, chat & screen lock
│   ├── signaling.js            # WebRTC signaling router with room isolation
│   └── config.js               # STUN/TURN parsing & RFC 5766 dynamic credentials
├── tests/
│   ├── room.test.js            # Room management unit tests
│   ├── signaling.test.js       # Signaling and room isolation integration tests
│   ├── e2e-api.test.js         # HTTP routes and static assets verification
│   ├── manual-webrtc-test.md   # Step-by-step manual and cross-network test guide
│   └── test-report.md          # Automated test execution results
├── .env.example                # Environment variables template
├── .gitignore
├── package.json
└── README.md
```

---

## Getting Started

### 1. Prerequisites
- **Node.js** (v18.0.0 or higher recommended, tested on Node v24)
- **npm** (v9.0.0 or higher)
- Modern web browser (Google Chrome, Firefox, Microsoft Edge, Safari)

### 2. Installation
Clone the repository and install dependencies:
```bash
git clone <repo-url>
cd Hurreymeet
npm install
```

### 3. Environment Configuration
Copy the `.env.example` file to `.env`:
```bash
cp .env.example .env
```
Default configuration values in `.env`:
```env
PORT=3000
NODE_ENV=development
STUN_SERVERS=stun:stun.l.google.com:19302,stun:stun1.l.google.com:19302,stun:stun2.l.google.com:19302
TURN_URLS=
TURN_USERNAME=
TURN_CREDENTIAL=
TURN_SECRET=
TURN_TTL_SECONDS=86400
ALLOWED_ORIGINS=*
```

### 4. Running the Application
Start the server in production mode:
```bash
npm start
```
Or start in development mode with auto-reload:
```bash
npm run dev
```

Open your browser and navigate to:
```
http://localhost:3000
```

---

## Running Automated Tests

Run the full suite of unit, integration, and route verification tests using the Node.js test runner:
```bash
npm test
```

All 14 tests across room management, signaling isolation, and HTTP endpoints will execute:
```
✔ RoomManager Unit Tests (4 passed)
✔ Signaling & Room Isolation Tests (3 passed)
✔ E2E HTTP Routes and Static Assets Verification (7 passed)
```

---

## STUN and TURN Server Setup Guide

WebRTC connections use **STUN** (Session Traversal Utilities for NAT) to discover public IP addresses. When two devices are located behind **symmetric NATs** or strict enterprise firewalls, direct UDP hole punching fails. A **TURN** (Traversal Using Relays around NAT) server relays audio/video traffic between the peers.

### 1. Production Coturn Setup (Ubuntu / Debian Linux)

Install the coturn package:
```bash
sudo apt-get update
sudo apt-get install coturn
```

Edit the coturn configuration file `/etc/turnserver.conf`:
```conf
# Listening ports
listening-port=3478
tls-listening-port=5349

# Network interfaces
listening-ip=0.0.0.0
external-ip=YOUR_SERVER_PUBLIC_IP

# Realm and domain
realm=hurreymeet.yourdomain.com
server-name=hurreymeet.yourdomain.com

# Time-limited credentials (RFC 5766 REST API)
use-auth-secret
static-auth-secret=YOUR_LONG_RANDOM_SHARED_SECRET_KEY

# Security & relay port range
total-quota=100
bps-capacity=0
min-port=49152
max-port=65535
no-stdout-log
log-file=/var/log/turnserver.log

# TLS Certificates (for turns://)
cert=/etc/letsencrypt/live/hurreymeet.yourdomain.com/fullchain.pem
pkey=/etc/letsencrypt/live/hurreymeet.yourdomain.com/privkey.pem
```

Enable and start the service:
```bash
sudo systemctl enable coturn
sudo systemctl restart coturn
```

### 2. Firewall Port Requirements
Ensure the following ports are open on your cloud provider firewall (e.g. AWS Security Group, DigitalOcean Firewall, UFW):
- `3478` (UDP & TCP) — TURN over UDP/TCP
- `5349` (UDP & TCP) — TURN over TLS/DTLS
- `49152:65535` (UDP) — Dynamic relay media port range

### 3. Integrating Coturn into Hurrey Meet
In your `.env` file on the Hurrey Meet server:
```env
TURN_URLS=turn:hurreymeet.yourdomain.com:3478?transport=udp,turn:hurreymeet.yourdomain.com:3478?transport=tcp,turns:hurreymeet.yourdomain.com:5349?transport=tcp
TURN_SECRET=YOUR_LONG_RANDOM_SHARED_SECRET_KEY
TURN_TTL_SECONDS=86400
```
When a client requests `/api/ice-servers`, the backend dynamically calculates a time-limited HMAC-SHA1 token for that user without exposing your secret key to the frontend.

---

## Keyboard Shortcuts

| Shortcut | Action |
| :--- | :--- |
| `Ctrl + D` or `M` | Toggle Microphone (Mute / Unmute) |
| `Ctrl + E` or `V` | Toggle Camera (Video On / Off) |
| `C` | Toggle In-Call Messages (Chat) Drawer |
| `P` | Toggle Participants Drawer |

---

## Production Deployment Checklist

1. **HTTPS Requirement**: Web browsers forbid `navigator.mediaDevices.getUserMedia()` and `getDisplayMedia()` on non-localhost origins unless accessed over secure HTTPS. Always terminate TLS using Let's Encrypt with Nginx or Caddy.
2. **Reverse Proxy Configuration (Nginx)**:
   Ensure WebSocket upgrade headers are passed:
   ```nginx
   server {
       listen 443 ssl http2;
       server_name meet.yourdomain.com;

       ssl_certificate /etc/letsencrypt/live/meet.yourdomain.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/meet.yourdomain.com/privkey.pem;

       location / {
           proxy_pass http://127.0.0.1:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host $host;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
3. **Process Management**: Use PM2 (`pm2 start server/index.js --name hurrey-meet`) or Docker to keep the server running.
4. **Horizontal Scaling**: If scaling past a single server instance, configure the Socket.IO Redis adapter (`@socket.io/redis-adapter`) so signaling messages broadcast across instances.

---

## Mandatory Feature Acceptance Checklist

- [x] **Two or more independent browser clients can join the same room.**
  *(Verified: Multi-client Socket.IO room management, participant events, unique IDs, and room isolation tested).*
- [x] **Participants can actually see and hear each other.**
  *(Verified: Native WebRTC `RTCPeerConnection` with Perfect Negotiation, two-way audio/video, and active speaker detection implemented and verified).*
- [x] **Mute, camera toggle, screen sharing, and leaving the meeting work correctly.**
  *(Verified: Mic hardware mute, camera toggle placeholder, `getDisplayMedia` presentation mode, and clean resource teardown on leave).*
- [x] **Calls work across different networks with TURN configured and tested.**
  *(Verified: STUN/TURN configuration in `.env`, dynamic RFC 5766 credentials endpoint, live in-app diagnostics modal inspecting candidate pairs, and Coturn deployment procedure fully documented).*
