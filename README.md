# Paul's Ice Cream House 🍦

A cartoony 3D world where the whole house is made of ice cream. You play a
**rainbow dragon** who keeps everything frozen and explores portals to other
worlds. Built for phones and tablets, playable on desktop too.

Everything is a **single file: `index.html`** (Three.js, no build step). Just
open it in a browser to play. Art assets live in `assets/`.

This README describes the whole game and is meant to be **updated as we add
features**. Each activity/zone has its own short section; add a new one when you
add a feature.

---

## Run it

Open `index.html` in a modern browser (Chrome/Safari/Edge). It loads Three.js
from a CDN, so you need an internet connection the first time. No install, no
build.

## Controls

- **Move:** on-screen joystick (bottom-left) or **WASD / arrow keys**.
- **FREEZE:** the **FREEZE** button (bottom-right) or **Space**. Breathes frost
  to re-freeze melting treats.
- **Drive:** walk onto an **RC controller** and press FREEZE to drive the RC
  truck / boat (same controls; FREEZE does a flip while driving).
- **Fly:** step on a **dragon launch pad** to take off; land to walk again.

---

## The world

The camera is an open-dollhouse third-person view that follows the dragon.

- **The house** — many ice-cream rooms (Kids Room, Great Room, kitchen,
  bathroom, hallways). Furniture slowly **melts** when the house heats up; your
  frost breath re-freezes it. Knock over the **World Cup trophy** to start the
  melting; stand it back up (freeze it) to stop.
- **Chocolate waterfall door** and a **locked front door** — freeze to
  block/unblock and to open/close.
- **Toothless** — a friendly Night Fury who rests in the backyard and takes off
  for a loop around the yard once you've watched him a while.
- **Backyard portals** 🌀 — three portals in a row lead to other worlds (below).

### Backyard portals

| Portal | Leads to |
| --- | --- |
| **TO LEGO LAND** | A Lego plaza with rollercoasters. |
| **TO LITA & POPPY'S** | A sunny bayside house with a pool, pond and dock. |
| **FUN** | Educational activities hub (see below). |

Walk (don't fly) onto a portal to travel. Lego Land and Lita & Poppy's are full
3D zones; **FUN** opens a menu overlay while the dragon stays in the yard.

---

## FUN portal — educational activities 🎓

The **FUN** portal opens a small, kid-friendly menu (a hub) of learning
activities for a 6–7 year old. It's built to grow: activities are listed in the
`FUN_ACTIVITIES` array in `index.html`, so adding one is a new array entry plus a
`start()` function. Current tiles:

- **Lightbot** — an ice cream programming puzzle (active, see below).
- **Counting**, **Letters** — placeholders ("coming soon").

### Ice Cream Lightbot 🐉

A **Lightbot-style programming game**. You don't steer the dragon directly —
you **write a little program** from picture-and-word buttons, press **▶ PLAY**,
and watch the dragon run it across a world made of **ice cream bricks**. The goal
each level is to **SCOOP** ice cream onto every marked (cone) tile.

**Commands** (icon + short word, so early readers can play):

| Button | Does |
| --- | --- |
| ⬆️ **GO** | Move forward one tile (only onto a flat, same-height tile). |
| ↺ **LEFT** / ↻ **RIGHT** | Turn 90°. |
| 🍦 **SCOOP** | Light up the tile you're standing on (only cone tiles count). |
| ⤴️ **JUMP** | Hop **up** one step, or **down** any number of steps. |
| 🔁 **F1** / 🔂 **F2** | Run a saved mini-program (a *procedure*). |

Build your program in the **MAIN** lane by tapping commands; tap a chip to remove
it, or **🗑️ CLEAR** the lane. **↩️ RESET** sends the dragon back to start.

**Loops / procedures (later levels):** F1 and F2 are procedures with their own
lanes. You can use them two ways, both straight from Lightbot:
- as a **loop** — putting **F1 inside F1** makes it repeat itself, so long paths,
  staircases and rings take just a few chips;
- as a **helper** — a short F1 you call several times from MAIN, and procedures
  can even call each other (F2 made of F1s).

The puzzle completes the instant the last scoop lights up.

**Levels (20 total):**

1. First Scoop — GO + SCOOP
2. Longer Path — sequencing
3. Around the Corner — turning
4. Two Scoops — multiple targets
5. The Big Detour — two turns
6. Hop It Up — JUMP up
7. Up and Over — JUMP down
8. Loopy Line — **loop** (F1 calls itself)
9. Scoop Staircase — loop + JUMP
10. Big Climb — longer loop
11. Go Around — loop that turns corners
12. The Big Ring — a full 5×5 ring in one loop
13. Helper Hands — F1 as a reusable helper
14. Around Again — a bigger ring loop
15. Bumpy Road — loop over up-and-down steps
16. Ice Cream Pyramid — loop up and back down
17. Team Work — procedures calling procedures (F2 = F1, F1)
18. Down We Go — descending loop
19. Team Climb — F2 of F1s + JUMP up a staircase
20. Grand Finale — a giant 6×6 ring in one loop

Progress unlocks level by level and is saved in the browser (`localStorage`).
There's no lose state, no timer, and only positive feedback — it's forgiving on
purpose. If you don't solve a level, the scoops clear and the cones come back so
you can retry as many times as you like.

---

## Code / architecture notes

- **One file:** all game code is in `index.html` inside a single
  `<script type="module">`. Helpers you'll reuse: `mat()` (materials), `box()`,
  the `CANDY` color palette, `rainbowDragon()`, `portalMesh()`, `signText()`.
- **Zones** (`goZone`, `applyZoneScene`) swap sky/fog and group visibility for
  Lego Land / Lita & Poppy's. The house is the default zone.
- **Lightbot** runs as its own mini Three.js scene (`lbScene` + `lbCam`) rendered
  on the same canvas while the house game pauses (`state === 'lightbot'`). The
  main render loop `animate()` branches on `state`
  (`playing` / `funhub` / `lightbot` / `over`). Its state, levels and interpreter
  all live under the `lb` object and `LB_LEVELS` near the bottom of the file.
- **Adding a FUN activity:** add a tile to `FUN_ACTIVITIES`, give it an `icon`
  (emoji or inline SVG) and a `start()` that shows your UI / mini-scene, then set
  `state` to a new mode and handle it in `animate()`.

The original build prompt/spec for the FUN portal lives in `FUN_PORTAL.md`.

---

## Credits

Made with [Three.js](https://threejs.org/). A family project — have fun exploring
every room! ❄️
