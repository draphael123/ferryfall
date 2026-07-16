# Credits

## Audio

**Gull cries** — `audio/seagull_ambient_*.wav`
Four "Seagull Ambient" recordings from **Solo Seagull Sound Effects** by
**Rango Mango**, via OpenGameArt.
<https://opengameart.org/content/solo-seagull-sound-effects>
Licence: **CC0 1.0 (public domain)** — no attribution required, commercial use
permitted. Credited here anyway, because that's the decent thing to do and it
records the licence for anyone who forks this.

Everything else is synthesised at runtime with the Web Audio API and has no
licence attached:

- **Sea, wind, rain** — three filtered beds off one looped pink-noise buffer.
  Wind and surf literally *are* filtered noise, so synthesis is the correct
  technique rather than a budget substitute — and it lets the whole soundscape
  ride the storm value continuously, which a sample can't.
- **The score** — composed at runtime in Aeolian; the root moves with the season
  and the whole bed ducks away under a storm.
- **Event sounds** — ship landing, placing, and loss.

## A note on licences, for future assets

Much of OpenGameArt is **CC BY-SA**, not CC0. CC BY-SA is copyleft: using it
would attach share-alike terms to this game's audio. Only CC0 (or CC-BY with
attribution recorded here) should go into this repo, and only deliberately.
