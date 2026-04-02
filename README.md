# Nuclear Fusion Interactive — Technical README

A **five-stage**, **500×500 px** guided experience about solar nuclear fusion: text-driven scenes, optional **Sun zoom** styling, **drag-and-drop** hydrogen atoms into a fusion zone, then **CSS-driven** fusion and energy visuals. The implementation is a **single static file**—no bundler or dependencies.

**Live build:** [https://content-interactives.github.io/nuclear_fusion/](https://content-interactives.github.io/nuclear_fusion/)

Curriculum, placement, and standards copy live in [Standards.md](Standards.md).

---

## Stack

| Layer | Technology |
|--------|------------|
| Delivery | One HTML document |
| Markup / style | Semantic HTML + **inline `<style>`** (~550 lines) |
| Logic | **Inline `<script>`** (~300 lines), vanilla DOM APIs |
| Assets | None external; stars and rays generated in JS |

---

## File layout

```
index.html    # Entire app: CSS, five stage roots, script (state, drag, transitions, reset)
Standards.md  # Non-technical: CK-12, NGSS alignment, objectives (not required to run the app)
```

---

## DOM structure

- **`.container`** — Fixed `500px × 500px`, centered on `body`, dark radial background.
- **`#stars`** — Filled by `createStars()` with **50** absolutely positioned `.star` divs (random position, size, `animation-delay`).
- **`.progress`** — Five `.progress-dot` elements; `data-stage` 1–5. Classes `active` / `completed` mirror `currentStage`.
- **Stages** — `.stage` blocks `#stage1` … `#stage5`. Only one has `.stage.active` at a time (`opacity: 1`, `pointer-events: auto`).
- **`#navButton`** — Bottom-centered; label and visibility change per stage (`hidden` when drag phase is active).

### Stage map (logical flow)

| `currentStage` | `#stageN` | Role |
|----------------|-----------|------|
| 1 | `stage1` | Sun + hook question |
| 2 | `stage2` | Interior copy; `#sunInterior` gets `.visible` after timeout |
| 3 | `stage3` | Plasma background, **four** `#h1`–`#h4` `.hydrogen` atoms, `#fusionZone`, counter, drag UX |
| 4 | `stage4` | `#energyFlash`, `.emitted-electron` ×4, `#helium`; `triggerFusion()` sequencing |
| 5 | `stage5` | Smaller `#sun5`, `#energyRays` (16 `.ray` divs from `createEnergyRays()`), closing copy |

`goToStage(n)` toggles active stage, updates dots, and runs a **`switch`** for timeouts, button text, and `initDragAndDrop()` (stage 3 only).

---

## JavaScript behavior

### Global state

- `currentStage` (1–5)
- `atomsInZone` (redundant with derived count but updated for display)
- `hydrogenAtoms` — map of atom `id` → `{ element, inZone, originalPos }` after `initDragAndDrop()`
- Drag: `isDragging`, `draggedAtom`, `dragOffset`

### Pointer handling (stage 3)

- **`initDragAndDrop`** — Registers `mousedown` / `touchstart` on each `.hydrogen` in `#stage3`; document-level `mousemove` / `touchmove` / `mouseup` / `touchend`.
- **`drag`** — Positions atom with `left` / `top` in px, clamped to **0–450** (50 px margin vs 500 width to account for atom size).
- **`endDrag`** — Compares atom center to **fusion zone** center in viewport space; if Euclidean distance **&lt; 70**, marks `inZone`, adds `.in-zone`, **snaps** to one of four fixed cluster positions (`zonePositions`). When **four** atoms are in zone, adds `.ready` to `#fusionZone`, then **`goToStage(4)`** after 800 ms.

### Fusion sequence (`triggerFusion`)

Delayed `setTimeout` chain: flash + electron `.fly` classes → helium `.visible` → reveal nav (“See the Result”). Durations align with CSS keyframes (`flash`, `flyOut1`–`4`).

### Reset

From stage 5 button: restores atom positions from `originalPos`, clears fusion-related classes, helium, flash, electrons, interior visibility, counter; **`goToStage(1)`**.

---

## CSS notes

- Heavy use of **keyframes** (`twinkle`, `pulse`, `plasmaMove`, `orbit`, `zoneGlow`, `flash`, `flyOut*`, `rayPulse`) and **transitions** on opacity/transform.
- **`.sun.zoomed`** exists in CSS but is **not** toggled in the shipped script (possible leftover or hook for future use).
- Energy **rays** use shared `.ray` class with per-element `transform: rotate(...) translateY(-100px)` and staggered `animation-delay`.

---

## Running locally

Open `index.html` in a browser or serve the folder with any static file server (avoids any `file://` quirks for fonts or future assets):

```bash
npx serve .
```

---

## Deploying

Static hosting only (e.g. GitHub Pages). Ensure the site **`base` URL** matches the repository name if you use relative links elsewhere; this app uses **no** root-relative asset paths beyond the single page.

---

## Maintenance / extension

- To change **hit radius** or **snap grid**, edit the **`70`** threshold and **`zonePositions`** in the script together with `.fusion-zone` size in CSS.
- New stages require a new `#stageN` block, a progress dot, and branches in **`goToStage`**, **`navButton` click**, and **`reset()`** if applicable.
- Drag listeners are added once when entering stage 3; **re-entering** stage 3 after reset without a full reload is not a current path (reset goes to stage 1). If you add mid-flow replay of stage 3, guard against **duplicate listeners** or remove them on leave.
