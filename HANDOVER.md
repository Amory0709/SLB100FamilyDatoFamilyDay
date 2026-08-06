# HANDOVER — SLB 100 Family Day

## TL;DR
Single-file static site. Three.js + Three.js GLB of an oil rig, served
via GitHub Pages. User tunes it visually by orbiting + clicking GUI
controls. **The whole project is `game.html`.** That's the only file
that matters.

## URLs
- **Live**: https://amory0709.github.io/SLB100FamilyDay/
- **Palette explorer**: https://amory0709.github.io/SLB100FamilyDay/palette.html
- **Repo**: https://github.com/Amory0709/SLB100FamilyDay
- **Default video asset** (loaded by `game.html`):
  `https://www.slb.com/static/anniversary28052026/assets/anniversity/video-home.webm`
  (transparent webm; rendered on a solid blue plate that fills behind it)

## Repo layout
```
SLB100FamilyDay/
├── index.html             ← mirror of game.html (added 9cf8356 — GitHub
│                            Pages serves whatever is index.html; without
│                            it, README.md would render as the landing page)
├── game.html              ← THE WHOLE APP (single source of truth; ~760 lines,
│                            all HTML + CSS + ES-module JS inline)
├── palette.html           ← macaron palette explorer (6 accent directions on #0014C8)
├── slb-style-oil-rig.glb  ← 3D model (~26 MB) — referenced by loader
├── HANDOVER.md            ← this file
├── README.md              ← stale (still describes pre-GLB step 1) — read HANDOVER
└── (no build step, no npm, no node_modules — pure static)
```

**Important**: any structural change to `game.html` (HTML/CSS/JS) should be mirrored
1-to-1 into `index.html` in the same commit. Pattern: edit `game.html`,
`cp game.html index.html`, then commit both. Don't let them drift.

## Tech stack
- **Three.js r160** via ESM import map → `unpkg.com`
- **lil-gui 0.19.2** for the lighting / camera / video control panel
- **GLB model**: `slb-style-oil-rig` (~26 MB), Y-up, identity rotation
  matrices throughout (THREE.GLTFExporter). Model added to scene with
  `model.rotation.y = 0` (helipad at −X).
- Tone mapping: `ACESFilmicToneMapping`
- Background: `#d1d1d1` (gray, chosen by user)
- No bundler, no transpiler, no service worker

## Current visual state (HEAD = `4213ab3`)
| Layer | Where | Z-index |
|---|---|---|
| **`#loader`** (SLB-blue full-viewport overlay) | covers entire viewport until GLB stream completes, then fades 500 ms and DOM-removed | 10000 |
| Loading text | `Loading… X KB / Y MB` — running byte counter, no percentage | inside loader |
| Loading bar | indeterminate slide L↔R (no percentage track) | inside loader |
| Loading spinner | white-on-blue ring above the bar | inside loader |
| 3D viewport | fills the page (oil rig model) | default |
| Oil rig | at origin, slow Y-axis spin (≈100 s / orbit, toggled by 🔄 自动旋转) | — |
| `#slb-panel` (right-side blue plate — also contains the video) | `fixed; top/right/bottom:0; width:28vw; min-width:360px; background:rgb(0,20,200)` | 25 |
| Video overlay | transparent webm, pinned to bottom of `#slb-panel` via `margin-top:auto`, `width:100%; max-height:56vh` | inside panel |
| × close button | inside panel, top-right — hides the entire panel (which hides the video with it) | 31 |
| ⚙ toggle button | inside panel, top-left — shows GUI; `display:none` after click | 32 |
| GUI (lil-gui) | hidden by default, `position:fixed; top:60; right:14`; **always on top once shown** | 9999 |
| `#hint` | bottom-left, "drag to orbit · scroll to zoom" | 10 |

After GLB loads, loader fades → P1 is restored then auto-rotate kicks in (default).

## GUI contents (top-level, in order)
- 📁 Background: `bgColor`
- 📁 Ambient / Hemisphere / Key / Fill / Rim / Environment: lighting
- 📁 🖼 视频: 水平 (videoX, 0..100 %), 垂直 (videoY, 0..100 %), 高度 (videoHeight, 100..900 px), 最大宽度 (videoMaxWidth, 20..100 vw), 背景色 (videoBgColor)
- ☐ 🔄 自动旋转: checkbox gating `model.rotation.y += dt * 0.06`
- 📁 📍 视角: 记下当前视角 / 回到 P1 / 清空序列 (HUD removed; pin data is still saved to localStorage and the GUI buttons still work — just no on-screen display)
- ↺ 重置全部: snaps every `params.*` back to defaults

## Camera behavior
- **Default view**: P1 — hardcoded
  - `cam = [-149.21, 147.49, 155.24]`
  - `tgt = [128.93, -14.50, -43.00]`
- **Persisted view**: `localStorage['slb100_cam_v1']` saves position +
  target on every `controls.change` event (debounced ~100 ms). On next
  load, the saved camera is restored if present. P1 is seeded first.
- **Pin view workflow** (📍 视角 folder; HUD was removed in `c031a49`):
  - "📍 记下当前视角" — appends current cam/tgt as P2, P3, P4, … to
    localStorage (no on-screen display; copy values from console.log)
  - "↩ 回到 P1" — instantly returns to hardcoded P1
  - "🗑 清空序列" — clears user-added pins (P1 is auto-reseeded)

## Auto-rotate
- **Per-frame `model.rotation.y += dt * 0.06`** in `tick()`. Speed is
  hardcoded — change that constant to tune.
- Toggleable via `params.autoRotate` checkbox in the GUI (default: true)
- Mouse-drag still overrides OrbitControls on top of auto-spin.
- `c51510d` removed the prior `controls.autoRotate` (camera orbit) —
  the spin is now on the model itself, not the camera.

## Things you can break — don't
1. **`fitToObject` is fine** — but user gives camera values to override
   it for P1. Pattern: `camera.position.set(...)` +
   `controls.target.set(...)` AFTER fitToObject, BEFORE the first save.
2. **Model natural orientation**: helipad is on `-X` (viewport left at
   default). Commit `dea6668` flipped it 180° and was **wrong** — keep
   that commit in history but never re-apply rotation.
   `model.rotation.y = 0` preserves the natural orientation on GLB load.
3. **GUI must stay hidden by default** — user explicitly said
   "把controller先隐藏". Use the ⚙ button to reopen for debugging.
4. **`<video>` is transparent-overlay only** — no `controls`,
   no `box-shadow`, no `border-radius`, `background:transparent`,
   `pointer-events:none`. User got furious when it looked like a player
   with a black background.
5. **The video and its blue plate must stack correctly.**
   `#slb-video` needs `position:relative; z-index:1` — otherwise the
   `position:absolute` blue plate (its DOM sibling inside the wrap)
   paints over the `position:static` video and hides it. `8b11008` is
   the fix commit; do not regress. The ×/⚙ buttons keep their
   `z-index:31/32` and still sit on top.
6. **GUI z-index must stay 9999** — `ddebd2f` lifted it above the video
   overlay (z25) and its buttons (z31/32). Lowering it lets the video
   cover the control panel when the 🖼 视频 sliders move the video over
   the panel area. (The lil-gui title bar is draggable, so the user can
   also just move the panel out of the way.)
7. **`game.html` and `index.html` must stay byte-identical** — the
   GitHub Pages entry is `index.html`. After every JS/CSS/HTML change,
   copy `game.html` → `index.html` in the same commit. (See "Repo
   layout" above.)
8. **The loader bar MUST NOT show a percentage.** GitHub Pages on
   `.glb` files does NOT emit `Content-Range` for `Range: bytes=X-` probes
   (verifiable with `curl -sI -H 'Range: bytes=0-0'` — it returns
   `200 OK` + `Content-Length`, not `206 Partial Content` + `Content-Range`).
   We tried in `924bf99` and the denominator mismatched the actual stream,
   making the bar hit 500 %+. **Final solution in `cb57add`**: bar is
   permanently indeterminate (L↔R slide), text shows only bytes received
   — no % sign, no `/`, no total. Don't reintroduce a percentage.
9. **Loader z-index 10000 + `display:flex; align-items:center`** — keeps
   it pinned to the viewport even during the brief moment before the
   WebGL canvas is sized. Don't change to absolute positioning over a
   small target.
10. **Loader fade-out order matters**: after `applyModel(gltf)` runs in
    `requestAnimationFrame`, wait one more frame so the first render
    *actually has pixels* before starting the opacity transition. Doing
    it earlier shows a black flash.

## Workflow expectations (learned the hard way)
- **User orbits with mouse and wants THAT angle remembered.** Don't try
  to guess the right `fitToObject` offset.
- **User prefers explicit GUI sliders for visible tweaks.** When
  changing anything visible on screen, surface it in the panel. Examples
  established this session: 🖼 视频 folder (`f703e72`), 🔄 自动旋转
  checkbox (`844242c`).
- **The video has alpha** — confirmed transparent.
- **"push一下" usually means "I think you forgot to push"** — always
  check `git status` first; if already in sync, just confirm and remind
  the user to hard-refresh (`Cmd+Shift+R`).
- **Hard refresh is mandatory.** GitHub Pages caches aggressively; tell
  the user to `Cmd+Shift+R` (or `Ctrl+F5`) after every commit.
- **"绕自身的Z轴" probably meant Y** when the model is Y-up. The GLB is
  Y-up (THREE.GLTFExporter, identity rotation matrices), so a literal
  Z-axis rotation makes the rig rock forward/backward. If the user asks
  for "spin" they almost always mean a turntable around vertical
  (= local Y) and not around the depth axis (= local Z). Confirmed by
  `a88783a` over `c51510d`.
- **When a visual change isn't quantifiable (e.g., spinner reaching 500 %),
  ask *which* failure mode they saw with a Telegram poll — A/B/C options
  beats an open "what's wrong?" question** (used in #2355 after the first
  loader bar attempt).
- **Don't promise per-cent accuracy if the server doesn't help you.** If
  the server's headers can't supply a denominator, drop the percentage
  entirely rather than show a wrong one. (Applied in `cb57add`.)
- **Edits to `game.html` structural files always mirror to `index.html`
  in the same commit.**

## Local dev
- No build step. Just serve the folder:
  ```bash
  cd SLB100FamilyDay
  python3 -m http.server 8790
  # open http://127.0.0.1:8790/index.html  (or /game.html — both work)
  ```
- Local server check: `curl -sI http://127.0.0.1:8790/index.html`
- Verify JS syntax: extract the inline `<script type="module">` block to
  `/tmp/check.mjs` and run `node --check /tmp/check.mjs`. No imports
  needed for syntax-only validation.
- Useful probes for understanding what GitHub Pages actually returns:
  - Headers in general: `curl -sI https://amory0709.github.io/SLB100FamilyDay/slb-style-oil-rig.glb`
  - Range probe: `curl -sI -H 'Range: bytes=0-0' …/slb-style-oil-rig.glb`
  - These were key to debugging the loader (see gotcha #8 above).

## Recent commits worth knowing about
```
4213ab3 style: anchor h1 to the bottom so the whole text block sits above the mode cards   ← HEAD
8796899 fix: anchor the mode prompt itself with margin-top auto
041b3aa fix: add no-cache meta tags so users see the latest deployment
8e6fada style: move 'Please select your mode' prompt above the mode cards
69b6e53 style: capitalise Welcome, push the heading up, anchor modes above video
bd17bf4 style: bump welcome heading size and add breathing room above mode prompt
d599178 feat: load SLBSans brand typeface via local @font-face
52a4b63 merge: bring remote main forward — GitHub web edits + my SLB blue work
cb57add fix: drop fake percentage, show only real bytes + indeterminate bar
b911956 tweak: loader background → SLB blue rgb(0,20,200) matching right panel
646211b tweak: shorten loader text 'Loading 3D scene…' → 'Loading…'
924bf99 fix: real-time byte progress on GLB load                                  [superseded by cb57add — gave 500%]
88baa84 feat: full-viewport loading screen with progress bar
9cf8356 fix: add index.html entry so GitHub Pages renders game.html content
3838354 tweak: english labels, video bottom, SLB font, panel padding, no video size cap
a88783a fix: spin model around Y (vertical) instead of Z (rocking)
c51510d feat: spin the model around its local Z axis (replaces camera turntable)
844242c feat: slowly auto-orbit the oil rig (turntable view)              [superseded by c51510d]
ddebd2f fix: GUI panel always on top once opened (z-index 20 → 9999)
8b11008 fix: video was hidden behind its own blue backdrop (stacking bug)
bdfb4ab feat: blue backdrop behind the video (same size, color in GUI)
f703e72 feat: add '🖼 视频' folder in GUI — video position + size now adjustable
c031a49 chore: remove the two debug HUD overlays (tag pill + pin-list)
be757f9 fix: bigger video, vertically centered on right edge
95ea820 fix: drop blue header, put the video in the top-right (much bigger)
a7821e4 fix: actually style the SLB header (blue bar + centered transparent video + close/toggle buttons)
0da75d5 docs: handover note — single-file Three.js + GH Pages project
093b7a8 feat: blue header bar + video on header + hide GUI               [superseded by 95ea820]
1539929 fix: video as transparent GIF-like overlay (not a player)
63cbeb4 feat: overlay SLB anniversary video (top-right, autoplay)
c5ac01a feat: gray background + hardcode P1 as default view
6e8b7f2 feat: pin view button — record a sequence of camera positions
f1a71cd feat: persist camera orbit position via localStorage
dea6668 style: rotate model 180° around Y — flip helipad to view-left  (WRONG, kept in history)
```

## Known open questions (don't act without checking with user first)
- Whether the auto-rotate speed (~0.06 rad/s, ≈100 s / orbit) is right
  for the demo. Speed is hardcoded in `tick()`.
- Whether `videoBgColor` should stay in the 🖼 视频 GUI folder
  (currently does) or be hidden once the user settles on a color.
- Whether the model should auto-rotate by default (`autoRotate:true`)
  or only when the user enables it (currently default-on; flip the
  literal to `false` in `params` if you want default-off).
- Whether to expose `model.rotation` axis choice (Y vs Z) as a GUI
  dropdown, or hardcode Y. Currently hardcoded to Y.
- Whether the loader should ever switch back to a percentage display
  (e.g., on a CDN that *does* send Content-Range). Currently always
  byte-counter + indeterminate — clean and unambiguous, but the user
  may eventually want a real progress bar if the GLB grows much past
  26 MB.
- Whether Single Mode / Couple Mode buttons should do anything other
  than log to console. Currently they only set a `.selected` class.
- Whether the lil-gui debug panel's text should be restyled to SLB blue
  (`#0014c8`). Currently uses the default dark theme; only visible when
  the user explicitly clicks ⚙. Touching it requires overriding several
  CSS custom props (--text-color, --title-text-color, --string-color…).
  Leave as-is until the user asks.

## Misc
- Untracked folder `outbound/` is an OpenClaw runtime temp dir — do
  not commit, do not delete without checking.
- Telegram rich messages are DISABLED for this bot. Use standard HTML only.
- **Text color rule** (from `3d8a401`): the canonical SLB blue is
  `#0014c8` / `rgb(0,20,200)`. Every readable piece of page text pulls
  from one color now — `#hint`, `#tag`, mode-card text, × / ⚙ glyphs,
  and (intentionally) the same value sits behind the right panel and
  loader as their background. **Text on SLB-blue surfaces stays white**
  — do not flip panel/loader text to blue "for consistency", that
  destroys readability. If the user later asks for literally-everything
  blue, change the loader/panel background to white first; otherwise
  the rule above holds.
- **Font stack (from `d599178`)**: body and all visible text now use
  `SLBSans` (no space, lowercase `b` — that's the actual @font-face
  family name). Two weights ship as `fonts/*.woff2`:
  - Regular (CSS 400) + Bold (CSS 500–700). Covers every weight value
    in the current page.
  - Other weights (Book/Light/Medium + italics) are NOT shipped.
  - The CDN on slb.com is the source but is unusable cross-origin
    (no `Access-Control-Allow-Origin`); local hosting is required.
  - **License caveat**: SLBSans is SLB's brand font. Local copies
    exist on the assumption that this site stays an internal SLB
    Family Day artifact. If it ever ships to an external audience,
    re-check the brand-asset license before publishing the woff2s.
