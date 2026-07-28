# HANDOVER — SLB 100 Family Day

## TL;DR
Single-file static site. Three.js + Three.js GLB of an oil rig, served via
GitHub Pages. User tunes it visually by orbiting + clicking GUI controls.
**The whole project is `game.html`.** That's the only file that matters.

## URLs
- **Live**: https://amory0709.github.io/SLB100FamilyDatoFamilyDay/
- **Repo**: https://github.com/Amory0709/SLB100FamilyDatoFamilyDay
- **Default video asset** (loaded by `game.html`):
  `https://www.slb.com/static/anniversary28052026/assets/anniversity/video-home.webm`
  (transparent webm, intended to overlay on the 3D viewport)

## Repo layout
```
SLB100FamilyDatoFamilyDay/
├── game.html              ← THE WHOLE APP (~430 lines, all in one)
├── HANDOVER.md            ← this file
└── (no build step, no npm, no node_modules — pure static)
```

## Tech stack
- **Three.js r160** via ESM import map → `unpkg.com`
- **lil-gui 0.19.2** for the lighting / camera control panel
- **GLB model**: `slb-style-oil-rig` (~26 MB), loaded at runtime from somewhere
- Tone mapping: `ACESFilmicToneMapping`
- Background: `#d1d1d1` (gray, chosen by user)
- No bundler, no transpiler, no service worker

## Current visual state (HEAD = `093b7a8`)
| Layer | Where | Z-index |
|---|---|---|
| Blue header bar | full width, top 0, height 110 px, `rgb(0, 20, 200)` | 25 |
| Video overlay | inside header, centered, autoplay/loop/muted/playsinline, transparent | 30 |
| ⚙ toggle button | header top-right, restores the GUI when clicked | 32 |
| × close button | hides the entire header (video + blue bar) | 31 |
| GUI (lil-gui) | **hidden by default** (`gui.hide()`), opens at top:60 right:14 | 20 |
| HUD `#tag` | top-left, `top:124px` (below header) | 10 |
| Pin-list HUD | top-left, below tag | 11 |
| 3D viewport | fills the rest of the page | default |

## Camera behavior
- **Default view**: P1 — hardcoded
  - `cam = [-149.21, 147.49, 155.24]`
  - `tgt = [128.93, -14.50, -43.00]`
- **Persisted view**: `localStorage['slb100_cam_v1']` saves position +
  target on every `controls.change` event (debounced ~100 ms). On next
  load, the saved camera is restored if present.
- **Pin view workflow** (button under "📍 视角" folder in GUI):
  - "📍 记下当前视角" — appends current cam/tgt as P2, P3, P4, …
  - "↩ 回到 P1" — instantly returns to hardcoded P1
  - "🗑 清空序列" — clears the user-added pins (P1 stays)

## Things you can break — don't
1. **`fitToObject` is fine** — but user gives camera values to override it
   for P1. Pattern: `camera.position.set(...)` + `controls.target.set(...)`
   AFTER fitToObject, BEFORE the first save.
2. **Model natural orientation**: helipad is on the `-X` side (viewport
   left when viewing default). Commit `dea6668` flipped it 180° and was
   **wrong** — keep that commit in history but never re-apply rotation.
3. **GUI must stay hidden by default** — user explicitly said
   "把controller先隐藏". If you need it for debugging, use the ⚙ button.
4. **`<video>` is transparent-overlay only** — no `controls`, no
   `box-shadow`, no `border-radius`, `background:transparent`,
   `pointer-events:none`. The user got furious when it looked like a
   player with a black background.

## Workflow expectations (learned the hard way)
- **User orbits with mouse and wants THAT angle remembered.** Don't try to
  guess the right `fitToObject` offset.
- **User prefers explicit buttons over auto-features.** When adding a
  feature, ask whether they want a button.
- **The video has alpha** — confirmed transparent in latest test.
- **"push一下"** sometimes means "I think you forgot to push" — always
  check `git status` first; if already in sync, just confirm and remind
  the user to hard-refresh (`Cmd+Shift+R`).
- **Hard refresh is mandatory.** GitHub Pages caches aggressively; tell
  the user to `Cmd+Shift+R` (or `Ctrl+F5`) after every commit.

## Local dev
- No build step. Just serve the folder:
  ```bash
  cd SLB100FamilyDatoFamilyDay
  python3 -m http.server 8790
  # open http://127.0.0.1:8790/game.html
  ```
- Local server check: `curl -sI http://127.0.0.1:8790/game.html`
- Verify syntax: parse the inline `<script type="module">` block with Node
  (replace `import` lines with comments first).

## Recent commits worth knowing about
```
093b7a8 feat: blue header bar + video on header + hide GUI      ← HEAD
1539929 fix: video as transparent GIF-like overlay (not a player)
63cbeb4 feat: overlay SLB anniversary video (top-right, autoplay)
c5ac01a feat: gray background + hardcode P1 as default view
6e8b7f2 feat: pin view button — record a sequence of camera positions
f1a71cd feat: persist camera orbit position via localStorage
dea6668 style: rotate model 180° around Y — flip helipad to view-left  (WRONG, kept in history)
```

## Known open questions (don't act without checking with user first)
- Whether user wants to record more camera pins (P2, P3, …) beyond P1.
- Whether the blue header height / color is final (110 px / `rgb(0,20,200)`).
- Whether to add a "show header again" button (currently the × button
  hides the header permanently until page reload).

## Misc
- Untracked folder `outbound/` is an OpenClaw runtime temp dir — do not
  commit, do not delete without checking.
- Telegram rich messages are DISABLED for this bot. Use standard HTML only.