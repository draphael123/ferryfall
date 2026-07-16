# FERRYFALL — the flotilla (rebuild, 2026-07-16)

Full rebuild on Daniel's direction: the game is **mostly on the water**. Your town
is a floating raft-town. Terrain stays, and stays load-bearing.

## Pillars (interview-locked — don't drift)

1. **One lashed raft-town, founded on the shallows.** A single contiguous mass.
   *(Revised 2026-07-16 on Daniel's call — was "rooted to bank moorings". It is
   now entirely on water; see "The anchoring rule" below. The bank is no longer
   special: the river is.)*
2. **v0 proves draft-as-land + the fairway conflict.** Nothing else matters until
   this lands.
3. **Residents afloat, trade ashore.** Your flotilla has residents; land villages
   are the demand you reach by ferry.
4. **Moorings yes, drift as threat only.** Anchoring topology is in; nothing
   physically floats away yet.

## The thesis, in one line

**Depth is land.** Deep water is the only place a deep-draft structure can sit —
and it is also the only road your ferries have. Every barge you moor in the
channel is real estate stolen from your own fairway.

## The anchoring rule (one rule, no mooring posts, all on water)

> **A thing stands on its own if it can reach the bed. Otherwise it holds onto
> its neighbours.**

`ANCHOR_MAX = 1.0` — and it is deliberately **equal to `DRAFT`**. That equality is
the whole game in one line:

> **Where boats can't go, you can stand. Where you can stand, boats can't go.**

Anything drawing over 1.0 m *by definition* only exists in water over 1.0 m, so it
can never touch bottom and can **never** stand alone. This isn't enforced — it's
arithmetic. **Verified over ~40,000 placements: zero legal deep-draft placement
anywhere on the map can self-anchor.** The Wharf, Tea House and Bath House have
**0 legal positions at turn zero** (vs 472 walkway / 285 cottage spots), so the
game physically cannot let you start with a money-maker. You must found in the
shallows and reach out.

This replaced bank moorings, which were an arbitrary constraint doing a job the
river already does — and which made terrain load-bearing in the wrong way (four
posts I sprinkled, rather than the shape of the water). Founding now works at
every reach of the river.

## Why the squeeze is automatic (the whole design rests on this)

The shallows are at the banks; the deep water is mid-channel. You found on the
shallows and may only grow **contiguously**. Therefore:

- Cheap shallow-draft housing (0.5 m) hugs the shore shelf — harmless.
- Valuable deep-draft services (1.9 m) can ONLY be reached by extending your raft
  **out into the channel** — directly across your own ferry lane.

So the conflict is geometric, not a rule I enforce. Growth *is* congestion. The
player builds their own traffic jam and can see it from orbit with no UI.

## The loop

1. Land villagers want to visit your flotilla (tea, baths, trade).
2. Ferries bring them from village docks to your **Wharf**. You earn per visitor.
3. Fare per visitor = `BASE + Σ(open services)`. More services = richer visitors.
4. Services are deep-draft → they must go in the channel → the fairway narrows →
   ferries detour → round trips lengthen → villagers wait → **fewer visitors**.
5. Release the pressure by **buying another boat** (220) or **dredging a new
   district off the fairway** (~1,200) and building there instead.

Self-limiting curve: the thing that makes you rich is the thing that strangles
you. That's Cities Skylines' traffic spiral, made physical.

### VERIFIED (2026-07-16) — the bet lands

Same 4 Bath Houses, two placements, 2 boats, 20 min:

| placement | detour | contentment | visitors |
|---|---|---|---|
| in the channel (free) | 1.72 | **0 — collapsed, queue growing** | 196 |
| off-fairway on a dredged district (1,177) | 1.35 | 0.82 | 234 |

A 3rd boat also repairs it (0 → 0.86). So congestion is a real, priced decision.

### Correction: "dredge a bypass" was fiction

You **cannot** dredge a canal behind your own town. The raft must be contiguous
to reach deep water, so its own spine spans the shelf a bypass would need — leave
a gap for the canal and the far half of your town isn't lashed on any more. The
rule that makes the game work also forecloses the obvious escape.

The real dredging verb is **make new land off the road**: deepen a patch of dead
shelf to 1.9 m, lash it to the raft with walkways, and put the money-makers there.
You pay ~1,200 coin to *not* strangle yourself. Same cost centre, better meaning —
it's the Skylines move of zoning a new district instead of clogging downtown.

### Placement bands (measured at iz 45, seed 3)

| structure | draft | can float |
|---|---|---|
| Walkway | 0.25 | ix 57–84 — whole river incl. dead shelf |
| Raft Cottage | 0.50 | ix 60–84 — safe shelf housing |
| Tea House | 1.10 | **ix 66–82 — channel only** |
| Bath House | 1.90 | **ix 67–82 — channel only** |

Every money-maker is channel-only. There is no safe place to put one. That's the
design at its purest — protect these bands if the terrain is ever retuned.

Services also need **staff**, drawn from a global pool of residents in cottages.
So growth needs both: cottages (cheap, shallow, safe) and services (dear, deep,
dangerous). Cottages are the enabler, services are the risk.

## Structures (v0)

| type | size | draft | cost | notes |
|---|---|---|---|---|
| Walkway | 1×1 | 0.25 | 25 | connective tissue, reaches across the shelf |
| Raft Cottage | 2×2 | 0.50 | 140 | +4 residents (staff pool) |
| Wharf | 3×2 | 1.30 | 200 | ferry terminal; required before any line |
| Tea House | 2×2 | 1.10 | 260 | +6 fare, needs 2 staff |
| Bath House | 3×3 | 1.90 | 480 | +14 fare, needs 3 staff — forces the channel |

## Balance trap: congestion is free unless boats are the bottleneck

The single hardest lesson of this rebuild. v0.1's `SPAWN_EVERY = 19` gave one boat
so much slack that it ran a village at **0.94 contentment with an empty queue** —
so a **+32% detour changed nothing at all**. Visitors 46 → 46. The core mechanic
was inert.

A longer trip only costs you if trip time converts into throughput, i.e. if demand
sits near a boat's capacity. At `SPAWN_EVERY = 5` the same 4 bath houses take a
2-boat service from 0.82 to 0.

**Whenever you retune the map, boat speed, or structure sizes, re-verify that a
choked fairway still collapses a working service.** If contentment is high with an
empty queue, the game has no teeth no matter how good the detour number looks.

Corollary: **the squeeze is directional.** Your town only blocks villages on the
far side of it — a route that never passes the raft never detours. Don't test the
mechanic on a village that sits upstream of the wharf (this cost me a whole test
cycle reading detour 1.16 → 1.16 and thinking the meter was broken).

## Rules

- **Footprint**: every cell must be water, unoccupied, unfrozen, `depth ≥ draft`.
- **Anchored**: the footprint must touch a shore mooring, or an already-anchored
  structure. Anchoring is inductive at placement time.
- **Blocking**: occupied cells are removed from the navigable grid. Your town is
  an obstacle to your own boats. That's the point.
- Refusals must say **which** rule and **by how much** ("draws 1.9 m, this is
  1.2 m") — in a no-fail builder the refusal message is the teaching surface.

## Kept from v0.1 (verified, don't re-derive)

Depth-coloured water · dredging · A* routing · out-and-back lines with marks ·
contentment from worst-wait · seasons on `S` · `__dbg` seam · `OutputPass` ·
`computeVertexNormals` on hand-built planes · boolean `.visible` · shadow bias ·
`toneMapped:false` lanterns.

See `README.md` for the full trap list — read it before touching render code.
