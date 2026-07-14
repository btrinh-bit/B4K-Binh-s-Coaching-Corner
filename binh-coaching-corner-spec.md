# Binh Coaching Corner — Feature Specification & Design Brief

**Product:** 3D badminton coaching and visualization tool
**Audience:** Badminton coaches and content creators
**Form factor:** Single self-contained HTML file, zero backend, cross-platform (desktop, tablet, mobile)
**Design mandate:** Modern, premium, fresh, exciting — glassmorphism surfaces, dark indigo base, lime-to-teal gradient accents, Space Grotesk / Inter type pairing

---

## 1. Core Modules & Functionalities

### Module A — Court Rendering (3D)

| Feature | Description | Success Criteria |
|---|---|---|
| BWF-standard court | Full doubles court 13.4 m × 6.1 m with singles sidelines at 5.18 m width, short service line 1.98 m from net, doubles long service line 0.76 m from baseline, net at 1.55 m (posts) / 1.524 m (center) | All line positions within ±1 cm of BWF Statutes at model scale; court renders within 2 s of page load on a mid-range phone (e.g., 2022 Android) |
| Orbit & zoom | Single-finger / left-drag orbits camera; pinch / scroll zooms; two-finger drag pans | 60 fps orbit on desktop, ≥30 fps on mobile; camera never clips below floor plane |
| Court-only rotation (desktop) | Holding **right mouse button** rotates the court group only, leaving camera fixed — players, annotations, and shuttle rotate with the court so relative positions are preserved | Right-drag rotates court group; release stops rotation; no player displacement relative to court lines |
| Auto-framing camera | On load and on viewport resize/orientation change, camera fits full court via iterative bounding-box projection | Entire court visible with ≥5% margin on any aspect ratio from 9:21 portrait to 21:9 ultrawide |
| Stationary court guarantee | Moving a player never transforms court geometry; players are architecturally independent scene objects | Dragging any player produces zero delta in court mesh transform (assert in code) |

### Module B — Player Management

| Feature | Description | Success Criteria |
|---|---|---|
| Add up to 6 players | Add via panel button; hard cap at 6 with disabled state and count badge ("4/6") | 7th add attempt is impossible; UI communicates cap without error dialog |
| Avatar types | Human (male / female) and six animals: quokka, bear, koala, panda, elephant, penguin — low-poly, smiling faces | Each avatar < 3,000 triangles; selectable at add time and editable afterward |
| Team colors & jerseys | Per-player jersey color (team presets + custom); floating sprite number label (1–6) always camera-facing | Label legible at all zoom levels; color change applies in < 100 ms |
| Free movement | Drag any player anywhere on or off the court plane via raycast-to-floor | Player follows pointer within 1 frame; no snapping unless grid-snap toggled |
| Contextual cursor (desktop) | Cursor becomes a **hand icon** when hovering within a player's pick radius, signaling "draggable"; reverts to default/orbit cursor elsewhere | Hover detection via raycast every pointermove; cursor swap < 1 frame |

### Module C — Annotation System

| Feature | Description | Success Criteria |
|---|---|---|
| Arrow tool | Freehand-draw movement arrows on the court plane; arrowhead auto-oriented to stroke direction | Arrow renders as 3D geometry anchored to court coordinates |
| Line tool | Freehand lines for zones, splits, footwork paths | Same anchoring as arrows; selectable color and width |
| Notes | Text notes attached either to a court position (3D pin) or to the setup globally (notes panel) | Notes persist in presets; editable and deletable |
| Persistence under camera motion | All annotations are children of the court coordinate space, so rotate/zoom/orbit never displaces them relative to court lines | Rotate court 360° → annotations remain pixel-consistent relative to lines |
| Eraser / undo | Tap-to-delete individual strokes; undo last stroke | Undo stack ≥ 20 steps |

### Module D — Presets (Templates)

| Feature | Description | Success Criteria |
|---|---|---|
| Save full setup | One tap captures players (positions, avatars, colors, numbers), all arrows/lines, all notes, shuttle position, and court/environment colors | Save completes < 200 ms; named by user |
| Recall any time | Preset list with name, timestamp, and player-count chip; tap to restore entire scene | Restore is visually identical to saved state |
| Storage | `localStorage` with in-memory fallback; export/import as `.json` file for sharing | Works offline; JSON round-trips losslessly |

---

## 2. Player Interaction Flow (Add → Select → Customize)

**Add**
1. Tap **"+ Player"** in the control panel (mobile: bottom sheet; desktop: side panel).
2. A compact picker pops up: **Type** (Human ♂ / Human ♀ / Quokka / Bear / Koala / Panda / Elephant / Penguin) rendered as icon tiles, **Team color** swatches, and **Number** (auto-assigned next free 1–6, editable).
3. Confirm → avatar spawns at a sensible default (next free service-court position) and the picker dismisses automatically.

**Select**
- Tap/click a player → soft lime glow ring at its base + a floating mini-toolbar (recolor, renumber, change avatar, delete).
- Tap empty court → deselect, mini-toolbar disappears. Nothing persistent clutters the viewport.

**Customize**
- From the mini-toolbar: swap avatar type in place (position preserved), cycle jersey color, edit number, or delete (with 3-second undo toast rather than a confirm dialog).

**Move**
- Drag directly. Desktop shows the hand cursor on hover; touch uses press-and-drag with a subtle lift shadow so the user knows the player is "picked up." Court never moves during a player drag.

**Cap behavior:** at 6/6 the add button dims and reads "6/6 — court full."

---

## 3. Color System

**Court surface palettes** (applied from a "Court" tab in the panel, live preview on tap):
- **Tournament Green** `#1B5E43` court / `#F4F1EA` lines — BWF broadcast classic
- **Championship Blue** `#1E4FA3` / white lines — Olympic mat style
- **Coral Clay** `#C4574E` / white lines
- **Slate Night** `#2A2F45` / lime `#C8F542` lines — matches app branding
- **Custom:** HSL color picker for surface and line color independently

**Environment palettes** (floor surround, ambient fog/backdrop, lighting tint):
- **Arena Dark** (deep indigo `#12142B`, cool rim light) — default
- **Studio White** (light gray floor, neutral light) — best for screen-recorded tutorials
- **Sunset Session** (warm gradient backdrop, amber key light)
- **Custom** backdrop color

**Application flow:** Panel → Appearance tab → two rows of swatch chips (Court / Environment). Tapping a chip applies instantly with a 300 ms cross-fade; the active chip shows a checkmark. Court line geometry always re-derives contrast automatically: if a chosen surface color yields < 3:1 contrast with the line color, the app nudges lines to white or near-black.

---

## 4. Annotation System Detail

**Tools:** Arrow, Line, Eraser, Note — presented as a slim floating tool rail (left edge desktop; a horizontal strip above the bottom sheet on mobile) that appears only when the "Draw" mode toggle is on and disappears otherwise.

**Drawing model:** While a draw tool is active, pointer input is captured for drawing and orbit is suspended (a small "Drawing — tap ✕ to exit" pill reminds the user). Strokes are raycast onto the court plane and stored as ordered court-space coordinate arrays, then rendered as extruded tube/ribbon geometry parented to the court group.

**Persistence under rotation/zoom:** because annotations live in court coordinates (not screen space), any camera orbit, zoom, or right-click court rotation carries them perfectly with the court lines. There is no re-projection step and no drift.

**Notes:**
- **Pinned notes:** tap the Note tool, tap a court location → small 3D pin with a billboard label (first ~24 chars) that expands to full text on tap.
- **Setup notes:** a free-text field in the panel for overall drill instructions; included in every preset.

**Styling options per stroke:** 4 colors (lime, white, coral, cyan), 2 widths. Kept deliberately minimal so coaches annotate fast mid-session.

---

## 5. Preset Saving Mechanism

**Format:** JSON, versioned.

```json
{
  "schema": "bcc-preset-v1",
  "name": "Mixed doubles rotation drill",
  "createdAt": "2026-07-14T09:30:00+10:00",
  "app": "Binh Coaching Corner",
  "players": [
    { "id": 1, "avatar": "panda", "color": "#C8F542",
      "number": 1, "pos": [2.1, 0, -3.4], "rotY": 1.57 }
  ],
  "shuttle": { "pos": [0.4, 0.9, 1.2] },
  "annotations": {
    "strokes": [ { "type": "arrow", "color": "#C8F542",
      "width": 2, "points": [[x, z], ...] } ],
    "pins": [ { "pos": [1.0, 0, 2.2], "text": "Cover cross drop" } ]
  },
  "notes": "Rotate after every 3rd lift.",
  "appearance": { "court": "slate-night", "environment": "arena-dark" }
}
```

**Storage & retrieval:**
- Saved to `localStorage` under a namespaced key (`bcc:presets`), with an in-memory fallback when storage is unavailable.
- Preset browser: card list showing name, relative timestamp, player-count chip, and a tiny top-down thumbnail (generated from a canvas snapshot at save time). Tap → full scene restore; long-press/right-click → rename, duplicate, delete, **export .json**.
- **Import:** file picker accepting `.json`; schema-validated before load, with a clear message if the version is unknown.

---

## 6. Accessibility

- **Contrast:** all UI text on glass panels ≥ 4.5:1 (WCAG AA); court palettes enforce ≥ 3:1 line-to-surface contrast automatically (see §3). Selection glow uses both color and a shape cue (ring) so it doesn't rely on hue alone.
- **Keyboard (desktop):** `Tab` cycles players; arrow keys nudge the selected player 10 cm (with `Shift` = 50 cm); `R` toggles draw mode; `1–4` select draw tools; `Ctrl/Cmd+Z` undo; `Ctrl/Cmd+S` save preset; `Esc` deselects/exits mode. All panel controls are focusable with visible focus rings.
- **Screen readers:** control panel is standard semantic HTML with ARIA labels ("Player 3, panda, team lime, position mid-court left"); an ARIA live region announces actions ("Player 4 added", "Preset saved"). The 3D canvas carries a text alternative summarizing current setup, regenerated on change.
- **Touch targets:** minimum 44 × 44 px for all controls; player pick radius generously padded on touch devices.
- **Motion sensitivity:** respects `prefers-reduced-motion` — camera transitions become instant cuts, cross-fades shorten.

---

## 7. UI Wireframe (High Level)

**Mobile / tablet (primary):**
- Full-bleed 3D viewport; portrait-optimized default camera angle.
- **Bottom sheet** control panel, collapsed by default behind a floating "Controls" pill; expands to ~45% height with tabs: **Players · Draw · Appearance · Presets · Notes**. Swipe down or tap outside to dismiss — nothing occludes the court when not needed.
- Contextual pop-ups (player picker, mini-toolbar, note editor) appear anchored near the touch point and auto-dismiss on outside tap.
- Top-right: minimal icon cluster — camera reset, undo, save.

**Desktop:**
- Same viewport-first layout; panel docks as a collapsible right sidebar (glass, 320 px).
- Left-edge floating tool rail in Draw mode only.
- Mouse mapping: **left-drag** orbit camera · **right-drag** rotate court only · **scroll** zoom · **hover near player** → hand cursor → drag to move.

**Visual hierarchy:** viewport dominates (>85% of screen at rest); panels use glassmorphism (blur + 1 px light border) over the dark indigo base; primary actions (Add Player, Save) use the lime-to-teal gradient; secondary controls are ghost-style. Space Grotesk for headings/numbers, Inter for body.

---

## 8. Technical Constraints & Recommendations

- **Rendering:** Three.js **r128** via CDN (already in use). Note r128 limits: no `CapsuleGeometry` (use cylinder + sphere composites), bundle `OrbitControls` logic manually or implement custom orbit — required anyway to support the right-drag court-rotation split.
- **Single-file constraint:** all CSS/JS inline; fonts from Google Fonts CDN; no build step, no external assets beyond CDNs.
- **Performance budget:** ≤ 50k triangles total scene; sprite labels instead of 3D text; single directional + ambient light; shadow map ≤ 1024 px on mobile.
- **Input abstraction:** unified Pointer Events layer so mouse, touch, and pen share one code path; mode arbitration (orbit vs. drag-player vs. draw vs. court-rotate) resolved by a single interaction state machine to prevent gesture conflicts.
- **Persistence:** `localStorage` + JSON export/import; feature-detect and fall back to in-memory with a non-blocking notice.
- **Compatibility target:** last 2 versions of Chrome, Safari, Firefox, Edge; iOS 15+; Android 10+.

---

## 9. Explicitly Out of Scope (v1)

Multiplayer/real-time sync, video export, physics-simulated rallies, backend accounts, and AR mode. Keeping these out preserves the zero-dependency single-file promise.
