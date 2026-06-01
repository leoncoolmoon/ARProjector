# AR Tracing Projector

A mobile-first PWA that overlays a reference image onto your phone's camera view, so you can trace drawings onto paper without a physical projector.

---

## Deployment

Requires HTTPS to access camera and sensors. Recommended free hosts:

- **Netlify** — drag the project folder into [app.netlify.com](https://app.netlify.com), done.
- **Vercel** — same drag-and-drop workflow.
- **GitHub Pages** — push to a repo and enable Pages in settings.

Once deployed, open the URL on your phone. The browser will offer "Add to Home Screen" — accept it for a fullscreen, app-like experience with no browser UI.

---

## Files

```
index.html     Main app (self-contained, includes WebGL shaders)
manifest.json  PWA metadata (name, icons, display mode)
sw.js          Service worker — caches assets for offline use
icon-192.png   Home screen icon
icon-512.png   Home screen icon (large)
```

---

## Usage

### Basic setup

1. Prop your phone above your paper — angled is fine.
2. Tap **📷 Camera** to start the camera feed (rear camera by default).
3. Load a reference image via **Local image**, a URL, paste (Ctrl+V), or drag-and-drop.
4. The reference image appears overlaid on the camera view. Drag to reposition, pinch to scale and rotate.

If multiple cameras are detected, tapping **📷 Camera** again cycles through them.

### Reference image controls (panel)

| Control | Description |
|---|---|
| Opacity | How transparent the reference image is |
| Scale | Size (also controllable by pinch gesture) |
| Rotate | Rotation (also controllable by two-finger twist) |
| ⇅ Swap layers | Toggle whether camera or reference image sits on top |
| ◑ Edge enhance | WebGL shader that darkens edges and brightens flat areas, making line structure clearer |
| ↔ H / ↕ V | Horizontal and vertical flip (top-right badges) |
| ↺ Reset position | Returns image to center at original scale |

### Perspective correction

Two approaches, use either or both together.

**Sensor-assisted (recommended):**
1. Lay your phone flat on the desk. Tap **① Lay flat & calibrate**. The sensor reads the baseline angle and stops.
2. Prop the phone at your drawing angle. Tap **② Prop up & capture**. The relative tilt is calculated and applied to Tilt X / Y automatically. The sensor stops immediately to save power.
3. Fine-tune with the **Tilt X** and **Tilt Y** sliders if needed.

**Manual:**
- Adjust **Tilt X** (left/right lean) and **Tilt Y** (front/back lean) sliders directly.
- For freeform distortion, tap **⬡ 4-corner warp** — four yellow handles appear at the corners of the reference image. Drag each corner independently.
- Tap **↺ Reset persp.** to clear all perspective settings.

### Chroma key (paper color removal)

When the camera layer is on top, the paper shows through as a bright area that can wash out the reference image below. Chroma key removes the paper color from the camera layer, replacing it with a flat fill so the reference image reads cleanly underneath.

1. Tap **🎯 Pick color**, then tap anywhere on the paper visible in the camera view. The sampled color is set as the key color.
2. Adjust **Tolerance** — higher values remove a broader range of similar colors.
3. Adjust **Feather** — softens the edge between transparent and opaque areas to reduce fringing.
4. The same color is automatically applied as the bottom-layer background fill, so the paper area still looks the same color visually.
5. Tap **✕ Disable** to turn chroma key off without losing the color settings.

**Tips:**
- Works best with colored paper (warm beige, light blue, grey) rather than pure white — white is harder to key without affecting highlights elsewhere in the frame.
- If the key is cutting into your hand or pencil, lower the tolerance.
- Re-pick the color if lighting changes significantly.

### Other controls

| Control | Description |
|---|---|
| ⛶ Full | Enter / exit fullscreen |
| Hide / Show panel | Collapses the control panel to maximize drawing area |

---

## Browser compatibility

| Feature | Chrome Android | Safari iOS |
|---|---|---|
| WebGL camera render | ✅ | ✅ |
| Chroma key shader | ✅ | ✅ |
| Camera switching | ✅ | ✅ |
| Focus lock | ✅ partial | ❌ |
| Device orientation sensor | ✅ | ✅ (requires user gesture) |
| WakeLock (screen-on) | ✅ | ❌ |
| PWA install | ✅ | ✅ (via Share → Add to Home Screen) |

On iOS, the screen may still sleep during long drawing sessions as WakeLock is not supported. Increase auto-lock timeout in Settings → Display & Brightness as a workaround.

---

## Technical notes

- The camera feed is rendered via a WebGL shader rather than a plain `<video>` element. This allows chroma key and edge enhancement to run on the GPU with minimal CPU load and battery usage.
- The device orientation sensor is only active for a single frame during each calibration/capture step, then immediately stopped.
- No data leaves the device. There is no backend, no analytics, no network requests after the initial page load.
