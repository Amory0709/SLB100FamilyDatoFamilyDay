# HANDOVER — SLB 100 Family Day

## TL;DR
Single-file static site. Three.js + Three.js GLB of an oil rig, served
via GitHub Pages. User tunes it visually by orbiting + clicking GUI
controls. **The whole project is `game.html`.** That's the only file
that matters.

## URLs
- **Live**: https://amory0709.github.io/SLB100FamilyDatoFamilyDay/
- **Repo**: https://github.com/Amory0709/SLB100FamilyDatoFamilyDay
- **Default video asset** (loaded by `game.html`):
  `https://www.slb.com/static/anniversary28052026/assets/anniversity/video-home.webm`
  (transparent webm; rendered on a solid blue plate that fills behind it)

## Repo layout
```
SLB100FamilyDatoFamilyDay/
├── game.html              ← THE WHOLE APP (~470 lines, all in one)
├── HANDOVER.md            ← this file
└── (no build step, no npm, no node_modules — pure static)
```

## Tech stack
- **Three.js r160** via ESM import map → `unpkg.com`
- **lil-gui 0.19.2** for the lighting / camera / video control panel
- **GLB model**: `slb-style-oil-rig` (~26 MB), Y-up, identity rotation
  matrices throughout (THREE.GLTFExporter). Model added to scene with
  `model.rotation.y = 0` (helipad at −X).
- Tone mapping: `ACESFilmicToneMapping`
- Background: `#d1d1d1` (gray, chosen by user)
- No bundler, no transpiler, no service worker

## Current visual state (HEAD = `a88783a`)
| Layer | Where | Z-index |
|---|---|---|
| 3D viewport | fills the page (oil rig model) | default |
| Oil rig | at origin, slow Y-axis spin (≈100 s / orbit, toggled by 🔄 自动旋转) | — |
| `#slb-video-bg` (blue plate behind video) | fills `#slb-video-wrap`, default `rgb(0,20,200)`, configurable via GUI | inside wrap |
| Video overlay | transparent webm, autoplay/loop/muted/playsinline, anchored at `(videoX%, videoY%)` of viewport with `translate(-50%,-50%)`, `max-height 480px / max-width 55vw` | inside wrap |
| × close button | inside video, top-right — hides the entire overlay | 31 |
| ⚙ toggle button | inside video, top-left — shows GUI; `display:none` after click | 32 |
| GUI (lil-gui) | hidden by default, `position:fixed; top:60; right:14`; **always on top once shown** | 9999 |
| `#hint` | bottom-left, "drag to orbit · scroll to zoom" | 10 |

After GLB loads, P1 is restored then auto-rotate kicks in (default).

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

## Local dev
- No build step. Just serve the folder:
  ```bash
  cd SLB100FamilyDatoFamilyDay
  python3 -m http.server 8790
  # open http://127.0.0.1:8790/game.html
  ```
- Local server check: `curl -sI http://127.0.0.1:8790/game.html`
- Verify syntax: parse the inline `<script type="module">` block with
  Node (replace `import` lines with comments first).

## Recent commits worth knowing about
```
a88783a fix: spin model around Y (vertical) instead of Z (rocking)   ← HEAD
c51510d feat: spin the model around its local Z axis (replaces camera turntable)
844242c feat: slowly auto-orbit the oil rig (turntable view)         [superseded by c51510d]
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

## Misc
- Untracked folder `outbound/` is an OpenClaw runtime temp dir — do
  not commit, do not delete without checking.
- Telegram rich messages are DISABLED for this bot. Use standard HTML only.
