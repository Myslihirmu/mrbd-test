# MRBD Test Suite

A single-file diagnostic web app for **Meta Ray-Ban Display** glasses. It exercises every
capability the glasses expose to a web app, navigable entirely with the five Neural Band keys.

## What it tests

| Screen | Tests |
|---|---|
| **Gesture & Keys** | All 5 inputs (`ArrowUp/Down/Left/Right`, `Enter`) with live per-key counters |
| **Motion / IMU** | Accelerometer (with & without gravity) + gyroscope rotation rate |
| **Orientation** | Tilt (bubble level) + compass heading (rose) |
| **GPS & Speed** | Lat/lon, accuracy, altitude, heading, device speed **and** a haversine-derived speed fallback (since `coords.speed` is often `null`) |
| **Audio Output** | Web Audio tone generator (variable frequency) + stereo auto-pan, to probe whether the glasses' speakers can be driven. Shows the live `AudioContext.state` — if it won't reach `running`, output is blocked by the sandbox |
| **Display & Color** | Safe-zone grid, text-size ladder, color bars, glare test, transparency test |
| **Storage** | `localStorage` persistence (launch count + counter) and `sessionStorage`, plus storage quota estimate |
| **Device Info** | Secure context, viewport, sensors/geolocation availability, environment |
| **Capabilities** ⚠ | Sub-menu of **gray-area probes** the device brief never confirmed — each flagged *UNVERIFIED on glasses* so an on-device result is read as a discovery (see below) |

### Capabilities sub-menu (the unconfirmed stuff)

Open **Capabilities** from Home, then pick a probe. `←` steps back out (Home → Capabilities → probe).

| Probe | What it answers |
|---|---|
| **Auto-detect** | Feature-detection table — does the API even *exist*? (≠ works on-lens) |
| **Haptics & Speech** | `navigator.vibrate` (feel it) and `speechSynthesis` TTS (hear it). `↑` vibrate, `↓` pattern, `Enter` speak |
| **Display & Lifecycle** | Screen Wake Lock toggle (`Enter`), live render FPS, and a log of visibility/focus/freeze events — i.e. what happens when you look away or the display sleeps |
| **Sensors & Battery** | Battery level/charging, Network Information, ambient-light sensor, and WebXR `immersive-ar` support. `Enter` to query |
| **Network Fetch** | Real `fetch` to same-origin / GitHub / Cloudflare with status, latency, bytes. `↑↓` endpoint, `Enter` run |
| **Raw Key Logger** | Logs **every** key (code, keyCode, repeat, press-duration) to reveal any undocumented Neural Band input. Exit with **double `Enter`** |

## Permissions to grant on the glasses

- **Location** — for the GPS & Speed screen.
- **Sensors (Motion)** — for the Motion/IMU and Orientation screens.

Nothing else is needed. The **Audio Output** test uses the Web Audio API, which needs **no
permission** — it just plays on the Enter press. (Audio *input* / microphone is not available on
the glasses, so there is no recording test.) The Location and Sensors prompts are triggered by
pressing **Enter** on their screen — they will not pop until you ask for them.

## Navigation

- **Home:** `↑`/`↓` move selection, `Enter` opens.
- **Most screens:** `←` returns Home; other keys do screen-specific things (shown in the footer hint).
- **Gesture screen exception:** `←` must stay testable there, so you exit it with a **double `Enter`** instead. It's the only overloaded key, and the footer says so.

## Architecture

Fully self-contained: one `index.html`, vanilla JS, **no backend, no API keys, no build step**.
All data comes from browser-native device APIs. This is why it deploys as a static site.

---

## Deploy to GitHub Pages (free, HTTPS)

GitHub Pages serves over HTTPS, which is **required** — `geolocation` and the motion sensors only
work in a secure context.

1. **Create a repo** at https://github.com/new (e.g. `mrbd-test`). Public is simplest.
2. **Push this folder.** Only `index.html` is required (`.claude/` and `README.md` are optional).
   ```bash
   cd /Users/kallehartiala/Desktop/GLASSESTEST
   git init
   git add index.html README.md
   git commit -m "MRBD test suite"
   git branch -M main
   git remote add origin https://github.com/<your-username>/mrbd-test.git
   git push -u origin main
   ```
3. **Enable Pages:** repo → **Settings** → **Pages** → under *Build and deployment* set
   **Source = Deploy from a branch**, **Branch = `main`**, folder **`/ (root)`**, then **Save**.
4. Wait ~1 minute. Your app is live at:
   ```
   https://<your-username>.github.io/mrbd-test/
   ```
5. **Open that URL on the glasses** (or in any desktop browser to test with the literal arrow keys).
   On first use of Motion/GPS, press **Enter** on those screens and accept the permission prompt.

### Local preview (optional)

```bash
cd /Users/kallehartiala/Desktop/GLASSESTEST
python3 -m http.server 4173
# open http://localhost:4173  (localhost counts as a secure context for sensors/GPS)
```

> Note: a desktop without an IMU/GPS will show `—` on the sensor screens and, after you press
> Enter, a "no data" message — that's expected; the readouts populate on the glasses/phone.
