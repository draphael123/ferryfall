# FERRYFALL

**Play: https://ferryfall.vercel.app** · repo `draphael123/ferryfall` (public)

Local: `python -m http.server 5774 -d ferryfall` (launch.json entry `ferryfall`).
Deploy: `vercel --prod --yes` **from PowerShell, not Bash**. The CLI prints its
version banner to stderr, so PowerShell reports NativeCommandError / exit 255
even on success — ignore it and read the output.

## Controls

`1` Build · `2` Dredge · `3` Line · `4` Break up · `5` Look · `S` season ·
`Esc` settings · `Enter` commit a line

Break up refunds in full, and refuses any cut that would set the rest of the raft
adrift — so you disassemble outside-in. Shelf pieces self-anchor and are always
freely removable; only chains out over deep water are load-bearing.

Saves: 3 slots on the title screen, 20 s autosave. The dredged bed is stored as a
diff against the generated one (dredging is the only thing that edits it), so a
save is a few hundred cells rather than 14,400 floats.

A cozy city-builder where the city is a river and everything arrives by boat.
Single-file `index.html`, Three.js via CDN, **port 5774** (`launch.json` entry `ferryfall`).

## Pillars (locked by interview 2026-07-16 — don't drift)

1. **3D tilt-shift**, Three.js. Cozy diorama, not a naturalistic landscape.
2. **v0 proves routing + boats.** Seasons are a manual `S` toggle, not a clock.
3. **Click docks in order.** You design the *stops*; the river decides the path (A*).
4. **Flat river, dredging only.** One water level. Elevation + locks are the first
   big feature *after* v0, not part of it.

No fail state. No money spiral. The pressure valve is **patience** — you're never
at risk of losing the city, only of letting someone down.

## The core design, in one line

`DRAFT = 1.00 m`. The main channel runs 2–3 m deep; every settlement sits up a
side-creek carved to **0.16–0.31 m**. So every village starts *visibly* reachable
and *actually* unrunnable, and the only way to reach anyone is to dredge a
channel you paid for with money the ferries earned.

**Colour the water by depth.** This is the whole UX thesis: pale sand = too
shallow, teal = navigable, dark = deep. "You cannot run this" is something you
*see* from orbit, not something you discover when a route silently fails.

## Economy (verified, don't re-tune by feel)

Opening: 900 grant → dock 120 → dredge ~480 → boat 220 = **820**. Your first
route costs almost the entire grant and leaves you 80 coin. Pays back in ~10 min;
by ~20 min you can afford village #2.

The decision that makes it a game (all measured over 40 min, stable):
**1 boat → 0.69 · 2 boats → 0.83 · 3 boats → 0.81.** The second boat (220) is
legibly worth buying; the third is waste. A whole new village costs 820. So the
money pushes you outward: *make the people you already serve happy, or go get the
people nobody serves at all.* Triage, not failure. The diminishing return on boat
#3 is load-bearing — it stops boat-spam being the answer.

## Balance trap: the queue must be stable, and only PATIENCE makes the gradient

A creek round trip is **~70 s measured** (not the ~55 s I estimated) carrying
`BOAT_CAP=4` = one villager per ~17.6 s.

- `SPAWN_EVERY` **must sit just under service rate** (19 s). Above it the queue
  grows without bound and contentment decays to 0 no matter how well you play —
  a broken tap. **6.5 s died in 4 minutes; 14 s looked stable at 10 minutes and
  was still quietly dying at 20.** Always check a balance claim at 40 min, not 10.
- But spawn rate **cannot** create the good/better gradient: at service rate the
  queue random-walks into decay, under it one boat trivially wins (19 s + the old
  `PATIENCE` gave 0.89 for one boat and the second boat bought nothing).
- So: keep the queue stable, and let **`PATIENCE`** decide what a wait is worth.
  That's the only safe lever.

## Rendering traps (each of these cost real time)

- **`OutputPass` is mandatory, not decoration.** A custom `ShaderPass` writes its
  shader output straight to the canvas with no tone mapping and no linear→sRGB
  encode. End the chain on a raw pass and the *entire game renders gamma-crushed
  to near-black* — which reads as "my lighting is wrong" and sends you tuning
  lights that were fine all along. Diagnose by comparing `composer.render()`
  against a direct `renderer.render()`; if direct is bright, it's the encode.
- **`watGeo.computeVertexNormals()`.** Build a plane by hand with no normal
  attribute and `MeshStandardMaterial` lights it with a zero normal → the river
  renders pure black.
- **`object.visible` must be a real boolean.** `draftLine && …` yields `null`, and
  three culls with `if (object.visible === false)` — a *strict* check — so `null`
  slips through and renders. Built docks kept showing their "not built" ghost.
- **Near-mirror `roughness` with no envmap reflects nothing** and reads black.
  Water sits at 0.3.
- **Bright `MeshBasicMaterial` blows out to white under ACES.** The lanterns set
  `toneMapped:false` — gold-vs-blue IS the contentment read.
- **Shadow bias.** A 2048 map over ~190 m is ~0.09 m/texel; without
  `bias`/`normalBias` the terrain acnes against itself and the valley reads muddy.

## Honesty traps

- **Ice is not shoal.** `navigable()` treats them the same; the *player* must not.
  One you dredge, the other you wait out. `berthWhy()` / `reachableIgnoringIce()`
  exist so a 1.49 m-deep frozen berth never says "too shallow". In a no-fail
  builder, the refusal message is the entire teaching surface — it cannot lie.
- **Contentment reads `max(oldest waiting, recentWait)`.** Reading only the
  current queue snaps it to 100% the instant a boat clears the jetty, so the
  lantern strobes once per round trip instead of reading route quality.
- **`recentWait` folds in the OLDEST boarder only** (`boarded[0]`; the queue is
  FIFO). Averaging every passenger in order converges on the *last* one processed
  — the newest arrival, who barely waited — so the lantern reports the luckiest
  villager on the boat. This game is about the other one.
- **Sites start `content: 0`**, not 1 — a HUD opening on "100% contentment" with
  0/4 served lies about the goal.

## Generation

`newValley(seed)` rolls a meander (two sines, never doubles back) + side-creeks,
then **rejects** any seed where a creek head isn't water-connected to the market
(`probeReachable()`). Dredging deepens water; it can't invent a channel across a
hill. Seed 7 rejects → lands on **10**.

## Verification seam — `window.__dbg`

v0 was verified without a human touching a mouse. Everything a click can do has a
headless twin: `dock(id)`, `openCreek(id)`, `line([ids])`, `dredgeLine(...)`,
`season(i)`, `sim(secs)`, `state()`, `path(a,b)`, `reseed(s)`, `grant(n)`,
`cam(...)`, `ts(...)`, `shot(q)`.

**But `__dbg` can hide fatal UX bugs from you.** `__dbg.dock()` worked perfectly
while the *real* Dock tool showed no rings at all — `setTool()` never called
`refreshGhosts()`. Always drive the real DOM buttons too.

Screenshots need the [[screenshot-pipeline]] memory (`computer` screenshot wedges
on this canvas) — capture hook → local upload server on a session-unique port →
`Read` the jpg.

## Next

- **Named villagers** (tension source #1, agreed): the dock-waiting system is
  already the hook they hang on. Merrin's the only one who needs the clinic.
- **Scarcity triage** (tension source #2): one dredging crew, three silting
  channels, six weeks to thaw.
- Elevation + locks. Then seasons on a real clock. A rare hundred-year flood last.
