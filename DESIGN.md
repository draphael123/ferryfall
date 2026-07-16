# FERRYFALL — open ocean (v0.3 rebuild, 2026-07-16)

Third design. Daniel: *"It's still a crossing. I want wide open oceans with
docks/etc."* — the earlier "it's still a river crossing" was a **complaint**, and
I misread it as agreement and built another round on top. Don't repeat that: when
he says "it's still X", X is the problem.

## Pillars (interview-locked)

1. **Open ocean, scattered shoals + a few islands.** Blue water to the horizon.
2. **Money is ships from the horizon.** Deep-sea traders sail in to your docks.
3. **Storms and exposure push back.**
4. Kept: the town floats, founded on the shallows, grown outward contiguously.

## The thesis

**You found on a shoal because it's the only place you can touch bottom. You
reach into blue water because that's the only place a real ship can reach you.
The shoal is also the only thing breaking the weather.**

So: *shelter and money point in opposite directions.* Every metre you reach out
buys a bigger ship and costs you cover. That's the same shape as the river
version (safe-and-worthless vs rich-and-dangerous) with the sea as the
antagonist instead of your own ferries — and it needs no channel, so nothing is
a crossing.

## Why the old squeeze is dead (and that's the bill)

The fairway conflict only worked because the channel was a narrow ribbon your
town could span. In open water boats always go around, so building can never
strangle traffic. **Depth-as-land survives; "your town is its own traffic jam"
does not.** Exposure replaces it.

## The loop

1. Found pontoons on a **shoal** (depth ≤ `ANCHOR_MAX` = 1.0 — you can pile it).
2. Chain outward. Out past the shoal nothing touches bottom, so each piece holds
   the next — exactly the v0.2 anchoring rule, unchanged.
3. Place a **dock**. Its class is set by the water depth it sits in:
   Jetty (2.5 m → coasters, small money) · Trader Dock (4.5 m → traders) ·
   Deep Berth (7.0 m → clippers, big money).
4. Ships spawn at the map edge and sail in — they need `depth ≥ their draft`, so
   shoals and breakwaters are real navigation obstacles.
5. **Storms.** Wind direction is set by season; strength too. Damage to a
   structure = `strength × exposure`. At 0 integrity it **breaks loose and is
   gone** — the drift threat, finally real.
6. **Exposure = fetch.** March upwind from a cell; if you hit a shoal, island or
   breakwater you're sheltered; if you reach open sea you're fully exposed. So
   the **lee side** of your shoal is safe — but the seasons rotate the wind, so
   no spot is safe forever.
7. **Breakwaters** are the release valve: rubble mounds founded on the bed (up to
   4 m), immune to weather, that block fetch. Build your own lee. This is what
   real ports do, and it's the ocean's answer to "dredge a district".

## Structures

| type | size | needs depth | cost | notes |
|---|---|---|---|---|
| Pontoon | 1×1 | 0.25 | 30 | connective tissue |
| Raft Cottage | 2×2 | 0.50 | 150 | +4 residents (staff pool) |
| Breakwater | 3×1 | ≤4.0 (founds on bed) | 240 | blocks fetch, storm-proof |
| Jetty | 2×2 | 2.5 | 180 | coasters · 1 staff |
| Trader Dock | 3×2 | 4.5 | 380 | traders · 2 staff |
| Deep Berth | 3×3 | 7.0 | 720 | clippers · 3 staff |

Cottages are shallow, safe and are the *enabler*; docks are deep, exposed and are
the *risk*. Same enabler/risk split that worked in v0.2.

## What v0 must prove

**That reaching out is a real dilemma** — a Deep Berth should earn several times a
Jetty and be genuinely likely to be destroyed if you don't shelter it. If a fully
exposed Deep Berth survives storms comfortably, the game has no teeth; if it can
never survive, it's a trap. Verify BOTH ends.

## Carried over — do not re-derive

Anchoring rule · depth-coloured water · demolish w/ adrift-guard · tutorial ·
settings · save slots (bed diff vs `H0`) · `__dbg` · `OutputPass` (mandatory) ·
`computeVertexNormals` on hand-built planes · boolean `.visible` · shadow bias ·
`toneMapped:false` emissives · path length must sum real step distances.
See `README.md` for the full trap list.

**Retired:** ferry lines, village contentment, the fairway/detour readout. The
code stays but no longer drives the game.
