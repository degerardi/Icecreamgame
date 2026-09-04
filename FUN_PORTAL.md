# FUN Portal — Educational Activities (spec / Claude Code prompt)

This is a build prompt for Claude Code. Paste it in, or point Claude at this file,
to add a **third backyard portal labeled FUN** that opens a small menu of
educational activities for a 6–7 year old. The first activity is an ice cream
themed, Lightbot-style programming game.

The whole app lives in one file: **`index.html`** (Three.js, ~2600 lines). There is
no build step. Open `index.html` in a browser to run it. Keep everything in that
one file unless a new asset is genuinely needed.

---

## 1. Context: how portals and zones already work

Read these spots in `index.html` before writing code:

- **Backyard bounds:** `rooms.push({ name:'Backyard', minX:2, maxX:28, minZ:44, maxZ:60 })` (~line 925).
- **Portal mesh + sign:** `portalMesh()` and `signText(text, bg)` (~lines 1221–1238).
- **Existing two backyard portals** (~lines 1239–1249):
  - `backyardPortal` at `(14, 0, 50.5)` → sign `TO LEGO LAND`, zone `'lego'`.
  - `litaPortal` at `(22, 0, 50.5)` → sign `TO LITA & POPPY'S`, zone `'lita'`.
  - Both push their `disc` into the `portalDiscs` array (animated in the render loop).
- **Zone scene setup:** `applyZoneScene(z)` sets `legoGroup.visible` / `litaGroup.visible`, sky, and fog (~line 1789).
- **Zone travel:** `goZone(z)` teleports the player/RC-truck and calls `showRoom(...)` (~line 1796).
- **Portal step-on detection:** the render loop checks `Math.hypot(player.pos.x-X, player.pos.z-Z) < 1.4` and calls `goZone(...)` (~lines 2405–2411). Flying over a portal does NOT trigger it — the player must land and walk on.
- **Room name toast:** `showRoom(name)` (~line 2216).
- **Palette:** `CANDY` object (~line 141) — reuse these colors (e.g. `CANDY.rViolet`, `CANDY.lemon`, `CANDY.mint`, `CANDY.straw`, `CANDY.choc`).

The three backyard portals should sit in a row along `z = 50.5`. LEGO is at `x=14`,
LITA at `x=22`. **Place the FUN portal at `x=6`** (west of LEGO, still inside the
backyard bounds).

---

## 2. Design decisions (already made — build to these)

1. **Presentation:** *3D grid in-world, 2D controls.* The robot/scoop moves across a
   real 3D tile grid rendered in the scene; the child builds the program in a **2D
   HTML panel overlaid** on top of the canvas. (Do NOT build a separate walk-around
   3D zone like Lego Land, and do NOT make it a pure 2D canvas puzzle.)
2. **FUN portal target:** a **small activity menu** (a hub screen), not straight into
   the game. The menu lists activities as tiles; the ice cream game is the first tile.
   Structure it so more activities can be added later with minimal changes.
3. **Command style (age 6–7):** **icons + short words.** Each command button shows a
   picture AND a short label (e.g. ▲ `GO`, ↺ `LEFT`, ↻ `RIGHT`, 🍦 `SCOOP`). Keep
   reading load light; the icon carries the meaning, the word reinforces it.

---

## 3. The FUN hub

When the player walks onto the FUN portal:

- Pause the 3D game input (like a modal) and show a **FUN menu overlay**: a friendly,
  candy-styled full-screen panel titled **FUN**.
- Show activity **tiles**. For now:
  - Tile 1: **"Ice Cream Robot"** (the programming game) — active.
  - Optionally 1–2 greyed-out "Coming soon" tiles (e.g. Counting, Letters) as
    placeholders so the layout reads as a menu, not a single button.
- A big **✕ / Back to Backyard** button closes the overlay and returns the player to
  the backyard (do NOT teleport to a new zone — the hub is UI, the player physically
  stays in the backyard).
- Keep the hub data-driven: an array like
  `const FUN_ACTIVITIES = [{ id, title, icon, enabled, start }]` so adding an
  activity later is one array entry + one `start()` function.

Portal wiring: add the FUN portal mesh + sign (`signText('FUN', CANDY.rViolet)` or
similar) next to the other two, push its disc into `portalDiscs`, and in the render
loop add a step-on check at `(6, 50.5)` that **opens the FUN overlay** instead of
calling `goZone`. Add a short cooldown (reuse the `portalCd` pattern) so it doesn't
re-open immediately when the overlay closes.

---

## 4. The game: "Ice Cream Robot" (Lightbot-style)

**Goal:** guide a little robot (or a walking ice-cream cone character) across a grid
of tiles to **scoop ice cream** onto every marked tile. When all target tiles are
scooped, the level is won.

### Board
- A small 3D grid of tiles rendered in the scene (start with **4×4 or 5×5**). Reuse
  candy colors and the existing lighting; tiles can be simple rounded boxes.
- Some tiles are **target tiles** (show a cone/marker or a glowing outline). Scooping
  on a target tile "fills" it (place a scoop mesh, play a little pop/scale animation).
- Later levels can add **raised tiles** requiring a JUMP command, but the first
  levels only need GO / LEFT / RIGHT / SCOOP.
- Show the robot facing a direction (N/E/S/W). Movement is one tile per GO in the
  facing direction. Turns are 90°.

### Commands (the 2D panel)
Bottom overlay panel with big touch-friendly buttons (icon + short word):
- ▲ **GO** — move forward one tile.
- ↺ **LEFT** — turn left 90°.
- ↻ **RIGHT** — turn right 90°.
- 🍦 **SCOOP** — place ice cream on the current tile (scores only if it's a target).

The child **taps commands to build a program sequence** shown as a row of icon chips
("your recipe"). Then:
- ▶ **PLAY** — runs the sequence step by step with a short delay between steps, and
  animates the robot in the 3D scene for each step.
- ⟲ **RESET** — clears the sequence and returns the robot to the start.
- Allow removing the last command (a **⌫ / Undo** chip) so kids can fix mistakes.

Keep it forgiving: no penalty for wrong moves, no timer. Running the program off the
grid just stops the robot at the edge (don't crash; maybe a gentle bump animation).

### Win / feedback
- When every target tile is scooped, show a big happy **"YUM! You did it!"** message
  with confetti/sprinkles and a **Next** button.
- Include **2–4 hand-designed levels** of increasing size/complexity (more targets,
  a turn required, then a longer path). Keep level data as a simple array of objects
  (grid size, start pos/facing, list of target tiles).

### Age-appropriateness
- Large buttons, high contrast, minimal text, no reading required to play.
- Positive-only feedback. No lose state, no score numbers that shame — just "targets
  left" as filled/empty cone icons if any progress indicator is shown at all.
- Sound is optional; if added, keep it soft and let it be muted.

---

## 5. Implementation plan (suggested order)

1. **Portal:** add the FUN portal mesh + `FUN` sign at `(6, 0, 50.5)`, push disc to
   `portalDiscs`, add step-on detection that opens the hub overlay.
2. **Hub overlay:** build the `FUN_ACTIVITIES` array + the FUN menu DOM/CSS overlay
   with tiles and a Back button. Wire portal → open, Back → close.
3. **Game shell:** clicking "Ice Cream Robot" hides the hub, builds the 3D grid, and
   shows the 2D command panel. Add a Back-to-hub button.
4. **Robot + movement:** render the robot, implement GO/LEFT/RIGHT/SCOOP as pure
   state changes, then animate them one step at a time on PLAY.
5. **Levels + win:** add level data, target tiles, scoop placement, win detection,
   the YUM screen, and Next-level flow.
6. **Polish:** animations (hop on GO, spin on turn, pop on SCOOP), sprinkles on win,
   touch sizing for tablets/phones.

### Constraints
- One file (`index.html`). Match the existing code style (concise, comment-annotated,
  `CANDY` palette, `mat(...)` helper, `THREE` primitives).
- The overlay must not break the main game: restore player input and hide the grid
  cleanly when returning to the backyard.
- Must work with touch (this is likely played on a tablet/phone) and mouse.
- No new heavy dependencies; reuse the Three.js already loaded.

---

## 6. Acceptance check
- A **FUN** portal is visible in the backyard alongside LEGO LAND and LITA & POPPY'S.
- Walking onto it opens a candy-styled **FUN menu** with an **Ice Cream Robot** tile.
- The game shows a **3D grid** with a robot and target tiles, plus a **2D command
  panel** with icon+word buttons.
- A child can build a sequence, press PLAY, watch the robot move/scoop, and see a
  **win celebration** when all targets are filled.
- **Back** buttons return cleanly to the hub and then to the backyard, with the main
  3D game fully playable again.
