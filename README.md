# Red Panda Randomizer

A randomizer and full parameter editor for the **Red Panda Particle 2** granular delay,
talking to the pedal directly over Web MIDI SysEx. One self-contained HTML file, no
dependencies, no build step.

The point is to find sounds you wouldn't have dialled in yourself — but *usable* ones.
A `musical` control keeps rolls inside bounds derived from the pedal's own manual, and
a `NO LIMITS` switch throws that away when you want it.

> **Not affiliated with, endorsed by, or supported by Red Panda.** Particle is their
> trademark. This is an independent tool built by reverse-engineering the SysEx
> protocol from the source map their web editor publishes. It only sends the same
> parameter and preset messages the official editor sends — it does not touch firmware.
> Use at your own risk; back up presets you care about before writing over slots.

## Run it

Web MIDI needs a secure context, so open it over localhost (not `file://`):

```sh
git clone https://github.com/jeremyblanchard/RedPandaRandomizer.git
cd RedPandaRandomizer
python3 -m http.server 8731
```

Then open <http://localhost:8731/> in **Chrome**, click **Connect**, and allow MIDI
(including SysEx) when prompted. Pedal connected by USB, powered on.

Close Red Panda's own editor tab first — two pages fighting over the same MIDI port
gets confusing, though it won't break anything.

## Using it

| | |
|---|---|
| **Randomize** (or `Space`) | Roll a whole new sound and send it to the pedal |
| **Mutate** (`m`) | Nudge the current sound by *amount* instead of replacing it |
| **musical** | How hard to keep the roll inside usable territory — see below |
| **character** | Biases the roll — see below |
| **◀ ▶** (`[` `]`) | Walk back and forth through every roll this session |
| **★ Keep** | Save the current sound to browser storage, recall it later |
| **lock** (square left of each row) | That parameter is never touched by Randomize/Mutate |
| **Sync ← Pedal** | Read all current values back off the pedal |
| **Panic** | Blend 50%, feedback 0, freeze 0 |

Any slider or dropdown also works as a plain editor — moving it sends immediately.

### The `musical` slider

Default 80. Uniform randomization sounds bad on this pedal for one structural reason
above all others: **Expert Mode is on, so every randomization source is live at once.**
Delay Rnd, Pitch Rnd, Detune and LFO Rate all run simultaneously no matter which Mode
you picked — four sources of chaos stacked on each other. On the stock pedal the Mode
switch neutralizes the ones it isn't using; in expert mode nothing does.

So the slider does two things. It **narrows each parameter's range**, and it
**neutralizes the motion parameters the chosen mode isn't about**, leaving each roll
with one clear idea:

| Mode | is about | everything else goes neutral |
|---|---|---|
| Pitch + Dens | Density, Pitch | Delay Rnd 0, Pitch Rnd 0, Detune ~0, LFO off |
| Pitch + LFO | LFO Rate, Pitch | " |
| Pitch + Detune | Detune, Pitch | " |
| Delay + Rnd | Delay Rnd, Delay Time | " |
| Delay + Pitch | Pitch Rnd, Delay Time | " |
| Delay + Rev | Direction, Delay Time | " |
| Delay + LFO | LFO Rate, Delay Time | " |
| Delay + Dens | Density, Delay Time | " |

It also snaps Pitch to consonant intervals (unison/octave/5th/4th/3rd/min7, weighted
toward the first three) instead of landing between notes, and couples the note
divisions — Chop is set to the same division as Delay or a shorter one, per the
manual's own advice, rather than rolling independently into polyrhythmic mush.

Not every rule matters equally, so they don't all reach full strength together. One
slider drives three curves:

| | rules | full strength at |
|---|---|---|
| **hard** | Freeze 0, blend ceiling, feedback ceiling | 40% |
| **mid** | motion neutralization, consonant pitch, Direction, Pitch Qnt | 80% |
| **soft** | curated feedback modes, coupled divisions | 100% |

Measured over 3,000 rolls per setting: unusable rolls (freeze engaged, too wet,
runaway feedback) hit 0% by 50%; mushy rolls (2+ competing motion sources, or off-key
pitch) hit 0% by 80%. Variety survives — even at 100% you still get all 8 modes, 13
pitch intervals, 25 division pairings and 5 feedback modes.

At 0 it behaves as it did before. **NO LIMITS** overrides it entirely.

### Musical ranges

The bounds the slider pulls toward, and why:

| Parameter | Musical | Reason |
|---|---|---|
| Blend | 25–60% | dry stays at unity to 50% — keeps a dry anchor under the effect |
| Feedback | 10–55% | recycle feedback stacks pitch and re-chops grains; runs away above this |
| Fdbk Tone | 45–90% | dark enough to sit behind the dry, bright enough to still be heard |
| Chop (grain) | 28–78% | very short grains give glitch tremolo and noise |
| Density | 55–100% | high = grains overlap into a continuous sound; low = disconnected blips |
| Freeze | 0 | it's a threshold — above 0 the pedal holds a buffer instead of your playing |
| Delay Time | 100–800 ms | of the full 0–2500 ms range |
| Delay Rnd | 0–22% | unless the mode is Delay + Rnd, then 25–85% |
| Direction | 100% fwd | mid values flip each grain at random; reverse belongs to Delay + Rev |
| Detune | 5–30% | unless Pitch + Detune, then 15–70% |
| Pitch Rnd | 0–15% | unless Delay + Pitch, then 20–65% |
| LFO Rate | off (noon) | unless an LFO mode, then it commits to one side rather than idling |
| Pan / Spread | near centre / 15–65% | full ping-pong on every grain is exhausting |
| Fdbk Mode | Auto, Post Delay, ±PP, Recycle, ±PP | the Repeat #/% family is inherently stuttery |
| Pitch Qnt | Semitones, 5th & Octave, Intervals, ±Inv | Free lands between notes |

### Characters

`Full chaos` · `Ambient wash` · `Glitch / stutter` · `Reverse` · `Time stretch` ·
`Pitch cloud` · `Shimmer / organ` · `Tempo-synced` · `Broken tape`

Each one constrains mode choice, parameter ranges, feedback mode and note divisions to
the region of the parameter space that actually produces that sound. Ranges came from
the Particle 2 manual's parameter descriptions — e.g. *Glitch* forces short grains,
high delay randomization and the `Repeat # / %` feedback modes; *Ambient* forces long
grains, high density, post-delay feedback and quantized pitch intervals.

### Constraints

- **NO LIMITS** — ignores the character *and* every other setting on this row. Each
  parameter is rolled uniformly across its whole legal range: all 8 modes, all 11
  feedback modes, all 7 pitch quantizations, every note division, and 0–100% on every
  continuous parameter. The controls it overrides grey out so you can see it's active.
  Locks still apply, so you can pin one or two things and let everything else go wild.

  Two things stay fixed even here, because moving them makes a roll *less* varied:

  - **Expert Mode stays on.** With it off, the pedal resets the previous mode's
    parameters every time Mode changes — it would silently overwrite most of the roll
    the moment it landed.
  - **The metaparameters are never sent.** Chop/Freeze, Delay/Pitch and Param are
    aliases for the individual parameters, and each overwrites several of them. Leaving
    them out is what makes the full space reachable, not a restriction on it.

- **max blend / max feedback** — hard ceilings on the two parameters that make a roll
  unusable rather than interesting.
- **allow freeze / drone** — off by default. The Freeze parameter is a threshold; roll
  it high and the pedal sits on a held buffer instead of passing your playing through.
- **snap pitch to semitones** — quantizes the Pitch parameter to the semitone grid.
- **firmware 2.2+ params** — turn off for firmware < 2.2 (Pan, Spread, Fdbk Mode,
  Pitch Qnt didn't exist yet).
- **always tempo-synced** — forces musical note divisions on delay/chop/density/LFO.

### Presets

`Write to pedal` saves the pedal's current edit buffer — i.e. the sound you just
rolled — into slots 1–127, optionally with a name. Slots 1–4 are the front-panel ones.
MIDI program 128 returns to live knob settings.

## Notes on how it drives the pedal

**Expert Mode is forced on for every roll.** Outside expert mode the pedal *resets the
previous mode's parameters* when you change Mode, which would wipe a roll. Messages are
also ordered Expert → Mode → everything else for the same reason.

**The three metaparameters are deliberately never randomized.** `Chop/Freeze` (35),
`Delay/Pitch` (36) and `Param` (37) are the front-panel knobs; each overwrites one or
more of the individual parameters depending on mode. This tool drives the individual
parameters (Chop, Freeze, Delay Time, Pitch, Detune, Density, Direction, …) directly,
which is the only way to get combinations the knobs can't reach.

## Protocol

Reverse-engineered from the official editor at
`redpandalab.com/content/apps/particle-editor/`, which ships a source map — the
original `rpl_midi.js` is recoverable from
`static/js/main.*.js.map`. The randomizer's output was diffed byte-for-byte against it.

```
F0 00 02 23 <family> <product> <msgver> <type> <propHi> <propLo> <data…> F7
   00 02 23 = Red Panda manufacturer ID
   family   = 0x01, product = 0x01 (Particle), msgver = 0x00
   type     = 0x34 get · 0x35 reply · 0x36 set
```

Parameter values are 24-bit: `trunc(value01 * 8388608)` sent as four 7-bit bytes,
MSB first. Property IDs with **bit 0x2000 set** carry a raw integer instead
(mode, feedback mode, note divisions, …) rather than a normalized value.

### Property IDs

| Parameter | ID | CC | | Parameter | ID | CC |
|---|---|---|---|---|---|---|
| Blend | 0 | 12 | | Mode | 8226 | 102 |
| Chop (grain size) | 1 | | | Freeze Mode | 8224 | |
| Freeze threshold | 2 | | | Delay Div | 8233 | |
| Feedback | 3 | 15 | | Chop Div | 8234 | |
| Fdbk Tone | 4 | 27 | | Density Div | 8235 | |
| Delay Time | 5 | | | LFO Div | 8236 | |
| Delay Rnd | 6 | | | Fdbk Mode | 8238 | |
| Pitch | 7 | | | Pitch Qnt | 8242 | |
| Detune | 8 | | | Expert Mode | 33 | 119 |
| Pitch Rnd | 9 | | | Trails | 45 | |
| Density | 10 | | | *Chop/Freeze (meta)* | 35 | 103 |
| Direction | 11 | | | *Delay/Pitch (meta)* | 36 | 104 |
| LFO Rate | 13 | | | *Param (meta)* | 37 | 105 |
| Pan | 20 | 32 | | Save preset | 0x7F13 | |
| Spread | 26 | 33 | | Program name | 0x7F1C | |

Note divisions: `0` = off, `1`–`9` = 8 bars … dotted whole, `10`–`31` = whole note …
1/128. Delay/Chop/Density accept 0 and 10–31; LFO accepts 0 and 1–26.

## Other Red Panda pedals

The repo is named broadly because the same protocol covers Red Panda's other pedals —
family/product IDs and the property map differ, but the framing, the 24-bit value
encoding and the `0x2000` indexed-parameter bit are shared. Adding Raster, Tensor or
Bitmap means a new parameter table and a new set of musical ranges, not new plumbing.
PRs welcome.

## License

MIT — see [LICENSE](LICENSE).
