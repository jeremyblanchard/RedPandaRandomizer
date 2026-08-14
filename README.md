# Red Panda Randomizer

A randomizer and full parameter editor for the **Red Panda Particle 2** granular delay,
talking to the pedal directly over Web MIDI SysEx. One self-contained HTML file, no
dependencies, no build step.

The point is to find sounds you wouldn't have dialled in yourself — but *usable* ones.
A `sanity` control keeps rolls inside bounds derived from the pedal's own manual, and
a `NO LIMITS` switch throws that away when you want it. Presets on the pedal are listed
in a sidebar you can load, name and overwrite from.

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
| **Randomize** (`Space`) | Throw away the current sound, roll every unlocked parameter from scratch |
| **Mutate** (`m`) | *Keep* the current sound and nudge every unlocked parameter by `amount`. Mode is held |
| **amount** | How far Mutate moves things. Boxed with Mutate because it has no effect on Randomize |
| **sanity** | How hard to keep the roll inside usable territory — see below |
| **history ◀ ▶** (`[` `]`) | Walk back through every roll this session and forward again |
| **★ Bookmark** | Store the current sound in this browser's Bookmarks list. Does not touch the pedal |
| **padlock** (left of each row) | Hold that parameter — Randomize and Mutate leave it alone |
| **Sync ← Pedal** | Read all current values back off the pedal |
| **⏻ Bypass** | Disengage the pedal (CC 81). Changes nothing about the sound, so re-engaging brings the same patch straight back. Follows the pedal's own footswitch |

Any slider or dropdown also works as a plain editor — moving it sends immediately.
Hover a parameter name for a description of what it does.

The toolbar is grouped by scope: whatever sits inside a bordered box is what the
labelled control acts on. `amount` lives in the dashed box with **Mutate** because it
affects Mutate alone; `sanity` sits in the outer box because it shapes Randomize and
Mutate alike.

Randomize and Mutate are coarse and fine. Measured on the default settings, Randomize
moves each parameter 36% of its range on average; Mutate at `amount` 25% moves it 2.7% and
leaves Mode alone. Randomize to find a neighbourhood, Mutate to search inside it.

### The intended loop

1. Hit **Randomize** (or tap `Space`) repeatedly while playing. Every roll goes straight
   to the pedal, so you hear each one immediately.
2. Something catches your ear — hit **★ Bookmark** before you roll past it.
3. Roll a few more. Went one too far? **history ◀** walks back through everything you
   rolled this session and **▶** walks forward again, so a sound is never lost to an
   itchy trigger finger.
4. Close but not right? **Lock** the parts that work — the square at the left of each
   row — and hit **Mutate** to jiggle only the rest by *amount*.
5. Keeper? **Write to pedal** puts it in a preset slot with a name.

**Locks are how you steer.** Lock Mode and every roll stays in one kind of sound —
lock it to Delay + Rev and you get nothing but reverse patches. Lock Mode *and* Delay
Time and you're auditioning grain and pitch behaviour over a fixed rhythmic bed; lock
everything except Pitch and Detune and you're auditioning harmony.

Locking Mode also redirects the neutralisation described below, so the roll stays built
around the mode you pinned rather than one it picked at random.

### The `sanity` slider

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

There was a `character` dropdown here (Ambient, Glitch, Reverse …). It is gone: sanity
lerps every range toward the musical one, so at any usable sanity setting almost nothing
of a character survived except which modes it allowed — and locking Mode does that more
directly.

### Sanity ranges

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

  Blend is the one exception: it never rolls below 1%, `NO LIMITS` included. At 0 the
  pedal is 100% dry, so the roll happened but none of it reaches your ears — that isn't
  a sound, it's a silent failure.

- **max blend / max feedback** — hard ceilings on the two parameters that make a roll
  unusable rather than interesting.
- **allow freeze / drone** — off by default. The Freeze parameter is a threshold; roll
  it high and the pedal sits on a held buffer instead of passing your playing through.
- **snap pitch to semitones** — quantizes the Pitch parameter to the semitone grid.
- **firmware 2.2+ params** — turn off for firmware < 2.2 (Pan, Spread, Fdbk Mode,
  Pitch Qnt didn't exist yet).
- **always tempo-synced** — forces musical note divisions on delay/chop/density/LFO.

### Presets

The sidebar lists all 127 pedal slots, the way the official editor's Preset tab does.
On connect it walks the pedal asking each slot whether it holds anything, then asks for
a name for the ones that do — so the list shows real names, not just numbers.

- **click a slot** — number or name — to load it, which also pulls its values back into
  the parameter list
- **✎** renames it in the pedal's flash
- **write** overwrites that slot with the current sound and offers to name it
- **Refresh** re-walks the pedal, e.g. after saving presets from the front panel

Presets 1–4 are the front-panel ones. The first row, **live**, returns the pedal to its
live knob settings. Overwriting an occupied preset asks first, naming what's there so
you know what you're about to lose.

#### A numbering wrinkle

The pedal counts programs from **zero**: wire program 0 is the first preset, 126 is the
last, and 127 is the live buffer. The manual and the front panel count presets from
one, and so does this list — so **preset _n_ is MIDI program _n−1_**, and the manual's
"MIDI program 128" is wire 127.

The official editor shows the raw 0-based number in its `#` column, which is why its
first row is `0`. If you are driving the pedal from a MIDI controller, use the program
number in each row's tooltip rather than the label.

**Bookmarks** below it are a different thing, and the distinction matters: ★ Bookmark
stores a sound in this browser only, so you can shortlist a dozen candidates without
burning through the pedal's 127 slots. Nothing is on the pedal until you hit **write**
on a preset row. Clearing browser data loses the bookmarks — Export writes them to JSON.

### Bypass

**⏻ Bypass** disengages the pedal with CC 81 rather than touching the patch, so
engaging it again returns the same sound. The pedal reports its own state on CC 88 when
you use the footswitch, so the button follows the pedal instead of drifting out of sync.

Bypass is not silence: your dry signal still passes through, unless the pedal's Bypass
Mode is set to Kill Dry. And with Trails on, existing repeats keep decaying after it.

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
