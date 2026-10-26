# forces-past-present-future

Interactive simulation for Richard Wright's Advanced Physics class, built from the course book's **4.2 Exploration: Past, Present, Future** (Unbalanced Forces Particle Model unit, `SAS_Physics_2026_2027 (16).pdf` in this folder, book pages 116–122). Each of the book's six scenarios is a snapshot of an object at constant velocity. Students tell the whole story: **PAST** (how it got to its current motion), **PRESENT** (balanced forces keep the velocity constant) and **FUTURE** (what would change it). The goal is to bridge from balanced to unbalanced forces by connecting the FBD and VAD to the motion map, especially **Δv⃗**: the result of the forces (ΣF⃗) points the same way as Δv⃗.

It reuses code from `physics-force-vad-solve` (see that repo's CLAUDE.md): the `wright vector` arrows, the label placer `makePlacer`, head-to-tail snapping with locked directions, Check my VAD (`checkChain`) and the vector addition animation. Keep shared code consistent when fixing it.

Live at **https://wrightphysics.org/forces-past-present-future/** (repo `WrightPhysics/forces-past-present-future`, GitHub Pages from `main`, root). The homepage card has not been added yet.

## How the project works

- Everything is in **one file: `index.html`**, with inline CSS and JS and no build step. The only outside dependency is Google Fonts.
- **Deploying = pushing to `main`** (GitHub Pages). Test with a cache-busting query string (`?v=2`).
- The homepage for wrightphysics.org is a separate repo, `WrightPhysics/wrightphysics.github.io` (one card per sim).
- Node isn't installed on this Mac, so `node --check` isn't available. Load the page in the browser preview and check the console instead. A quick full test: for every scenario and part, copy `storyPlaced()` into `buildMap()` and run `runCheck()`; all 18 must pass.

## Working with Richard

- He is a physics teacher and former consulting engineer. He gives feedback through screenshots; expect visual, precise requests.
- **Use his wording exactly** when he specifies text. Don't paraphrase it.
- Make the change asked for and nothing else; mention side effects briefly.
- Commit and push only when he asks.

## Page structure

1. Title **"Past, Present, and Future"**, a short intro line, and the scenario picker (← / dropdown / →), labeled "4.2 #N: name". The bar also has the **mode toggle**: **Build it myself** / **Show me the story**.
2. Top row: **Written Description** (book text verbatim + key) | **Sketch (the snapshot)** (sketch capped at 250 px tall).
3. **Watch It Happen** (Richard's request): the animation with its VAD beside it, and **the story graph (velocity vs. clock reading) right under the animation** (Richard wanted students to see the v–t graph with the animation). Details in its own section below.
4. Part tabs: PAST / PRESENT (the snapshot) / FUTURE, then that part's description. In story mode, a Balanced/Unbalanced pill is also shown.
5. Work row: **Motion Map and Δv⃗** and **FBD** (with force key) on the left; **VAD** on the right.

Every panel except the VAD has a Hide/Show button.

## Modes

- **Build it myself:** the same as the VAD-solve sim, but per part. Each part keeps its own VAD (`st.builds["si:ph"]`), so switching tabs doesn't erase work.
  - The FBD gives the part's forces (direction only until placed).
  - **F_g has a fixed length of `W_SQ` = 4 squares** and sets the scale. Every other force can be stretched.
  - Check my VAD has two steps: (1) every force joined head to tail; (2) consistent with the motion map. When Δv = 0 the VAD must end where it started. When Δv ≠ 0 it must not, and ΣF⃗ (from the start of the chain to its end) must point along Δv⃗ within `DIR_TOL` = 15°. Hints only, no "show me the answer".
  - A ✓ appears on the tab of each part that passed.
- **Show me the story:** the worked VAD (`storyPlaced`, chained in the `niceOrder` order and centered), the FBD drawn to scale, the ΣF⃗ arrow, and Δv⃗ inset.
  - Switching parts **tweens** the arrow lengths (`st.tween`, matched by force type and direction, so F_NCS→F_NCF morphs). Forces that disappear shrink away.
  - Playing the Watch It Happen animation steps through the three parts, and the VAD tweens at each boundary.

## Watch It Happen (the observation)

- **Layout:** `.wstage` is a grid. On desktop it's animation | VAD on top, then the graph and slider at full width. On phones (≤ 640 px) the order is animation, graph, slider, VAD, so the graph stays right under the animation.
- **Story graph** (`renderGraph`):
  - Split into PAST / PRESENT / FUTURE by dashed lines, like the book. The current part is shaded and its segment is drawn in `--dv`.
  - A clock cursor (line + dot) shows `st.T`. Clicking or dragging on the graph moves the clock. The direction of motion in the snapshot is positive.
  - `graphLayout()` picks a wide, short viewBox (900 wide) or the taller phone one (490 wide) from the graph's width.
  - The slider is inset to the graph's time axis (`GX0`/`GX1`), so its handle lines up with the cursor.
  - A "Velocity vs. clock reading graph" checkbox hides it (for students sketching Part A); the PAST/PRESENT/FUTURE labels under the slider then appear.

- **One clock, `st.T`, from 0 to 3** (past, present, future; `PLAY_MS` per part). Everything follows it through `setT()`: the animation, the slider, the graph cursor, and the part tab (`goPhase` when T crosses a boundary).
  - Picking a tab moves T to the middle of that part.
  - Changing scenario sets T = 1.5 (the snapshot).
  - Play from the end restarts at 0.
- **Velocity comes from the story graph** (`vAt`, piecewise linear). Position is its integral (`posAt`), so the motion is consistent with the graph and the forces.
- **`WATCH[i]`, in the same order as `SCN`**, defines each scene:
  - `e`: screen direction of + motion.
  - `vb`: the scene's viewBox crop.
  - `ctr(x)`: the object's center at position x.
  - `draw(T, x)`: the scene at that moment. These are the story events:
    - Puck: your hand pushes, then stays where it let go; the air cushion stops in the future.
    - Block: the finger stops and lifts off.
    - Child: the sidewalk (made longer at Richard's request; belt 60–470) has marks that move at belt speed. The child steps off its end at the end of the present.
    - Parachutist: the parachute opens during the first 0.3, and the figure is drawn at 1.6×.
    - Elevator: Floor 4 down to the Lobby.
    - Car: the cable snaps at T = 2, a "snap!" note appears, and the truck drives on; the car coasts up to a stop at T = 3.
- **Overlays:**
  - Motion map dots (default on): dots at equal clock intervals of 0.2.
  - Velocity arrow (default on): light green, drawn beside the object, reversing when v < 0.
  - Forces on the object (default off): the current part's forces from the object's center, at 42 px per weight.
- **VAD for this part** (default on, Richard's request): a small VAD (`#wvad`, `renderWatchVAD`) beside the scene for the part the clock is in. It wraps below the scene on narrow screens.
  - In Show me the story, it's the worked VAD, with ΣF⃗ and the tween between parts.
  - In Build it myself, it mirrors **the student's own VAD** for that part, so it never reveals the answer. Until they build one, it says "Build your … VAD below". ΣF⃗ appears there only when "Show ΣF⃗ and Δv⃗" is on; otherwise it's the dashed gap.
  - Its labels use `makePlacer`.
  - `renderVAD` and `renderWatch` both refresh it.
- The readout describes the motion at that instant from the actual velocity ("at rest for an instant"; ", moving backward" is added if a story ever has v < 0).
- **Clock not to scale** (Richard's request): notes under the controls and the story graph say each part gets equal time here, though in real life some parts are very brief (a landing, a cable snapping) and others are long.
- **Hands** (Richard flagged the first version as "robotic"):
  - `handFlat`: a flat hand pushing with the fingertips, used for the puck.
  - `handPoint`: a fist with the index finger out, used for the block in the animation and the sketch.
  - Both have tapered forearms with a cuff line.

## Connecting balanced and unbalanced (story mode)

- A tip line under the title (`.story-tip`, story mode only) tells students to press Play and look for the pattern.
- The part panel shows a cause-and-effect chain (`renderChain`):
  - Unbalanced: **FORCES** Unbalanced → **ΣF⃗** points <dir> → **Δv⃗** points <dir> → **MOTION** speeding up or slowing down, with "ΣF⃗ points the same way as / opposite to the way the object is moving."
  - Balanced: Balanced → ΣF = 0 → Δv⃗ = 0 → constant velocity.
  - Direction words come from `DIR_WORD` (right, left, up, down, up/down the hill).
- Switching to story mode turns on "Forces on the object" in the animation.

## Order of addition in worked VADs (Richard's request)

`niceOrder(phase)` tries every order and picks the easiest to read:
1. **Never add two opposite forces one after the other** when any order avoids it.
2. Keep turning the same way, so the VAD reads as one loop.
3. Start with F_g.

For example, the block becomes F_g down → push right → F_N up → friction left, a rectangle. Parts with every force on one line (elevator, parachutist, the puck's and child's present) can't avoid opposite neighbors; those keep the 8 px sideways nudge. Students still choose their own order in Build mode.

## ΣF and Δv (Richard's choice)

- **Default:** a gap is a dashed segment **with no arrowhead**, labeled ΣF ≠ 0, as in the VAD-solve sim. A closed VAD gets a green ring and ΣF = 0.
- **"Show ΣF⃗ and Δv⃗"** checkbox (build mode; always on in story mode):
  - The gap becomes a ΣF⃗ arrow in `--warn` with a filled stealth tip.
  - A small inset in the VAD's top right shows Δv⃗ "from the motion map", so students can see the two point the same way.
  - The status line adds whether they match.
- Status wording: "The VAD ends where it started, so the forces are balanced (ΣF = 0)." / "The VAD doesn't end where it started, so the forces are unbalanced (ΣF ≠ 0)." Never "closes."

## Scenarios and stories

- Book text is verbatim. The parachutist omits the book's hint sentence.
- The PAST and FUTURE stories were drafted by Claude and **approved by Richard** ("look good and seem appropriate"). He chose one fixed future per scenario.
- Force sizes are in units of the weight (`mag`), chosen so the story graph's slopes are consistent: equal |ΣF| in past and future wherever |Δv| is equal.

| # | Past | Present | Future |
|---|------|---------|--------|
| 1 Hover puck (moving left) | your hand pushes it from rest (P = 0.5 W) | F_g, F_N only | battery dies; floor friction slows it (0.5 W) |
| 2 Block + finger (right) | push 1.2 W > friction 0.6 W | push = friction = 0.6 W (3 N on 5 N) | finger lifts; friction alone |
| 3 Child on sidewalk (right) | sidewalk friction speeds him up | F_g, F_N only | steps onto floor; floor friction slows him (F_NCF, F_fCF) |
| 4 Parachutist (down) | chute opens: drag 2 W slows them (v from 3 to 1) | drag = weight | landing: ground N 0.9 W + drag 0.6 W |
| 5 Elevator (down) | T = 0.8 W, speeds up downward | T = W | T = 1.2 W, slows to a stop |
| 6 Car up 30° icy hill | T = 1.0 W (Richard asked for it to differ clearly from the present) | T = 0.5 W | cable snaps: slows to a stop for an instant, about to slide back (ΣF 0.5 W both ways, so the graph ends at 0) |

## Notation and drawing

- Type-On-By subscripts (`F_gPE`). Drag is `D`.
- Force colors are by type (`TYPE_COL`): g `--f0`, T/P `--f1`, N `--f2`, f `--f3`, D `--f4`.
- Velocity arrows are light green `--vel` with open barbs. Δv⃗ is dark green `--dv` with a filled stealth tip (`stealthArrow`).
- On the motion map, dot spacing and arrow length follow the speed (`SPEEDS`). Vertical motion maps run down the panel, with Δv⃗ to the right.
- Collinear arrows that double back in the VAD (F_N up after F_g down) are drawn nudged 8 px to their right-hand side so both stay visible. ΣF⃗ along the same line is nudged to the other side.
- In the FBD, forces that share a direction are drawn side by side (parachutist landing: D and N).
- HTML vector symbols use `<i class="vx">v</i>`, a CSS mask arrow in `currentColor`. Atkinson has no combining arrow (U+20D7), and it rendered as a box. SVG text uses `mathText`.
