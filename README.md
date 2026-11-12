# 🥊 Desi Showdown
### *Political Satire Fighter — 3D Browser P2P Fighting Game*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Three.js](https://img.shields.io/badge/threejs-r160-blue.svg)](https://threejs.org/)
[![WebRTC](https://img.shields.io/badge/WebRTC-P2P-brightgreen.svg)](https://webrtc.org/)
[![No backend required](https://img.shields.io/badge/zero%20server-P2P%20only-orange)]()
[![Works offline](https://img.shields.io/badge/works-mobile%20%7C%20desktop%20%7C%20PWA-ff69b4)]()

**Desi Showdown** is a 3D satirical political fighting game that runs entirely in the browser — no downloads, no accounts, no app stores. It's a self-contained, single-file HTML game with zero build process and minimal bandwidth requirements. Challenge friends to peer-to-peer multiplayer matches over WebRTC, or fight the CPU at three difficulty levels.

> Work of satirical fiction and parody. All characters are fictional caricatures for the purposes of humour and social commentary under freedom of expression. Not affiliated with or endorsed by any real person, political party, or organization.

![Desi Showdown Banner](https://repository-images.githubusercontent.com/876543210/abcdef-placeholder)

---

## ✨ Features
- 🎮 **Instant play** — single `index.html` file, opens directly in any modern browser
- 🥊 **Full fighting game mechanics**
  - Light punch, heavy kick, unblockable grab, screen-shaking SUPER move
  - Combo system: PPP, PPK, PKK, KK chains with cancel windows
  - Special moves: forward+punch (lunging heavy), down+kick (low knockdown sweep)
  - Blocking, jumping, crouching, hitstun, blockstun, invincibility frames
  - SHAKTI super meter that charges on hits/blocks
  - Best-of-3 rounds with 99-second timer, KOs, timeouts, and draws
- 🤖 **CPU opponent** with three AI difficulty levels (reactive distance-based state machine — no cheating, just smarter timing)
- 🌐 **Zero-latency P2P multiplayer** over WebRTC DataChannels
  - Firebase Realtime Database used *only* for temporary signaling (SDP exchange) — **zero gameplay data ever touches the server**
  - Rooms auto-clean themselves up: data is deleted the second peers connect, and `onDisconnect()` cleans up stale rooms
  - One-byte packed input over unreliable/unordered DataChannel for minimal latency (classic fighting-game rollback-ready netcode)
  - RTT/ping monitoring, automatic reconnection, rematch support
  - 4-character shareable room codes with WhatsApp / copy-link share
- 🎵 **Procedurally synthesized audio** via Web Audio API — no external sound files
- 📱 **Full mobile & touch support** with virtual joystick + buttons, landscape lock, and PWA install-to-home-screen
- ⚔️ **Cast of 9 satirical fighter characters** including a secret boss
- 📰 Satirical "Daily Jumla" headlines on every win screen
- 💾 Local storage for settings — no account required

---

## 🎮 Controls
### Desktop
| Action | Keys |
|---|---|
| Move left/right | `A` / `D` (or ← / →) |
| Jump | `W` or ↑ |
| Crouch | `S` or ↓ |
| Block | Hold back (away from opponent) |
| Punch | `J` |
| Kick | `K` |
| Grab | `U` (or J+K) |
| Super (when SHAKTI full) | `L` or `Space` |
| Pause | `Esc` |
| Mute | `N` |

### Mobile
- Virtual D-pad joystick on the left for movement
- Punch, Kick, Grab, Super buttons on the right
- Works best in landscape orientation

### Multiplayer
1. Click **PLAY WITH FRIEND**
2. Click **HOST A ROOM** — you'll get a 4-character room code
3. Share the code with your friend (WhatsApp button does this automatically)
4. Friend enters the code and clicks JOIN
5. WebRTC connects directly peer-to-peer and the room is deleted from Firebase instantly
6. All fight inputs travel directly between your browsers with no server in between

---

## 🏗️ How It Works (Architecture)

```
┌──────────────────────────────────────────────────────────┐
│  Your browser                                            │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Single-file vanilla JS SPA                        │  │
│  │  ├── Three.js WebGL 3D renderer + scene            │  │
│  │  ├── Frame-based fighting state machine (60fps)    │  │
│  │  │   • idle → walk → jump → crouch                 │  │
│  │  │   • attack states: windup → active → recovery   │  │
│  │  │   • combo chaining + cancel windows             │  │
│  │  ├── Web Audio API synthesized sound engine        │  │
│  │  ├── DOM HUD overlay (health, meter, timer)        │  │
│  │  └── Netcode layer:                                │  │
│  │       ├─ ctrl channel (ordered/reliable):          │  │
│  │       │     ping/pong RTT, rematch, session msgs   │  │
│  │       └─ input channel (unordered, maxRetransmits=0)│  │
│  │             • 1 byte/frame packed input bitfield   │  │
│  │             • no TCP head-of-line blocking         │  │
│  └───────────────┬────────────────────────────────────┘  │
│                  │ Only SDP signaling (~1KB total)       │
│                  ▼                                       │
│         ┌────────────────────┐                           │
│         │ Firebase Realtime  │  ◄── Stateless broker     │
│         │ /room_XXXX node    │      (deleted on connect) │
│         └─────────┬──────────┘                           │
│                   │ ICE candidates                       │
│         ┌─────────▼──────────┐                           │
│         │ STUN (Google)      │  ◄── NAT hole-punching    │
│         └─────────┬──────────┘                           │
│                   │                                      │
└───────────────────┼──────────────────────────────────────┘
                    │ Direct UDP-style P2P connection
                    ▼
┌──────────────────────────────────────────────────────────┐
│  Your friend's browser                                   │
│  ── Same code, no servers between us after handshake ──  │
└──────────────────────────────────────────────────────────┘
```

### Key design decisions:
1. **No build step, no frameworks**: The entire game is ~67KB of raw HTML/CSS/JS in one file. No npm install, no webpack/vite build required. Just open it in a browser.
2. **Three.js for rendering**: Low-poly procedural fighters, dynamic lighting, shadows, and camera shake — all without 3D model assets (fighters are built from primitives and animated procedurally).
3. **Zero-cost multiplayer**: Firebase's free tier can handle millions of room-creation handshakes per month because rooms live for ~1-5 seconds before being deleted, and all gameplay is offloaded to direct P2P connections. You can run this game forever for free.
4. **Netcode design**: The `input` DataChannel uses `ordered: false, maxRetransmits: 0` — exactly the same pattern used by AAA rollback netcode fighters like Street Fighter 6 and Guilty Gear Strive. Old input frames are never re-sent; only the latest input state matters, keeping latency as low as possible.
5. **Audio synthesis**: All punch, kick, hit, block, and KO sounds are generated on the fly with `OscillatorNode`s and `GainNode`s. No external audio assets needed, keeping the file tiny.
6. **Satire protection**: Built-in disclaimer gate on first launch, stored in localStorage.

---

## 🚀 Run It Locally
You don't *need* a server — because it uses ES module imports, you can either:
1. **Just open `index.html`** in a modern Chrome/Edge/Firefox/Safari (all CDN imports work directly), OR
2. Serve it locally with any static server:
```bash
# Python 3
python3 -m http.server 8080

# Node.js (serve)
npx serve .

# PHP
php -S localhost:8080
```
Then open http://localhost:8080 in your browser.

### To host it yourself
- Upload the single `index.html` file to any static host: Netlify, Vercel, Cloudflare Pages, GitHub Pages, S3, Neocities, literally anything that serves static files.
- No server-side code required. No backend to maintain.
- The default Firebase config in the file is for the public demo — replace it with your own Firebase project credentials if you want to run your own instance.

---

## 🛠️ Tech Stack
| Layer | Technology | Purpose |
|---|---|---|
| Rendering | Three.js r160 (via esm.sh CDN) | 3D scene, characters, lighting, shadows |
| Game logic | Vanilla ES2020 JavaScript | Frame-based state machine, combat, physics |
| Audio | Web Audio API | Procedural sound effect synthesis |
| Multiplayer | Native WebRTC RTCPeerConnection + RTCDataChannel | P2P gameplay transport |
| Signaling | Firebase Realtime Database (CDN) | Temporary SDP offer/answer exchange |
| UI | Hand-rolled HTML/CSS | HUD, menus, mobile touch controls |
| Styling | Raw CSS (no frameworks) | All UI styles inline |
| PWA | Web App Manifest | Add-to-homescreen, landscape lock |
| Build | **None** | Zero compilation, zero dependencies |

---

## 🎯 Reverse Engineering Note
This game is a clean-room implementation inspired by studying *Sansad Showdown / fightkaro.com* using the REA reverse engineering methodology:
- Passive static bundle analysis of the Vite chunks (`main-*.js`, `rooms-*.js`)
- Frame-timing, combat, and netcode patterns extracted from the minified source
- All original code written from scratch based on observed architecture, not copied
- All characters, art, audio, and assets are 100% original and procedurally generated
- The netcode architecture follows fightkaro's excellent open pattern (2-DataChannel P2P with long-poll signaling) but uses Firebase Realtime Database instead of a custom HTTP API

---

## 📜 License
MIT License — do whatever you want with it. Fork it, mod it, add more characters, port it to rollback netcode, make your own version. Satire belongs to everyone.

If you build something cool with it, tag me — I'd love to see it!

---

## 🤝 Contributing
PRs are welcome! Some ideas for extensions:
- More characters and stages
- Rollback netcode (GGPO-style) with input prediction
- Online leaderboards (add Cloudflare Workers + Durable Objects)
- More combo paths and special moves
- Soundtrack with procedural chiptune music
- Replays and match recording
- Controller / gamepad support
- More languages
- Different game modes (2v2, battle royale, tag team)

---

### Jai Hind 🇮🇳 Now go beat up some fictional political caricatures.
