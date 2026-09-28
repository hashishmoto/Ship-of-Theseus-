# Prompt: FX automation for the SoT siblings (DnB, jungle, dub, dubstep)

Copy everything below the line into a new Claude Code session on this repo.

---

You're working in the `hashishmoto/Ship-of-Theseus-` repo. It holds live Strudel engines that send MIDI (through `IAC Driver Bus 1`) into Ableton. The main engine is `sot 0.96` (Ship of Theseus, house/techno/psy/dubtechno). Your job is to give its siblings the same FX automation it now has:

- `.13 DnB` — the DnB engine, modes `dnb` and `dnbHalftime`. Jungle is not a mode yet. Adding it is part of the work, but talk to me first.
- `dnb. 13` — the Bass Engine, modes `trap`, `dub` and `dubstep`.

Read the FX section at the bottom of `sot 0.96` first (from `const MIDI_CHAN_FX` to the end). It is the reference implementation. Copy its structure; don't reinvent it.

## How I work (follow this)

- Keep answers short, in short sentences. Give one detail at a time, not numbered walls of text.
- Play devil's advocate. Point out the catch in my idea and in your own.
- Ask before big or creative changes. Fix plain bugs, but tell me first.
- Label every number as fact (researched), logic (derived), or guess. I confirm guesses by ear.
- Each version is a new file. Copy the file, bump the `vNN` in the header, and edit the copy. Never overwrite my last version.
- Commit and push after each agreed step. Give me the raw GitHub link to paste into Strudel.

## The rig (same for every engine)

- **Return A**: Replika XT, in an Audio Effect Rack with 16 macros.
- **Return B**: Raum, in a rack.
- **A→B send (CC 95)**: return A feeding B, for the dub chain. It's one direction only. Never add B→A, because it would make a feedback loop.
- Both Mix knobs stay fully wet (1), since they're on returns. Freeze is always 0.
- **All FX CCs go on MIDI channel 16.** Nothing plays on 16.

**Voice channels** — SoT uses perc 1, bass 2, stab/skank 3, lead 4, pad 5. The siblings differ: DnB has lead on 3, and the Bass Engine has lead 3 and skank 4. Propose moving them to the SoT layout. Ask me first, because I have to re-route tracks in Ableton.

**Sends** (per voice, per effect):

| Voice | Replika (A) | Raum (B) |
|---|---|---|
| perc | 85 | 90 |
| bass | 86 | 91 |
| stab/skank | 87 | 92 |
| lead | 88 | 93 |
| pad | 89 | 94 |

**Replika** — macro names as they appear on my rack:

| CC | Macro | CC | Macro |
|---|---|---|---|
| 117 | Feedback | 109 | LoCut |
| 118 | Mix | 110 | HiCut |
| 119 | Saturation | 111 | Flt Reso |
| 102 | Wow & Flutter | 112 | Fbk Loop |
| 103 | Flt Stereo | 113 | Feedback Balance |
| 106 | Flt Rate | 114 | Delay Time B |
| 107 | Filter Mode (stepped) | 115 | Flt Cutoff |
| 108 | Delay Time A | 116 | Flt Depth |

**Filter Mode zones**, measured on the rack (0–127): LP 0–25, HP 26–50, BP 51–76, Peak 77–101, Notch 102–127. Send mid-zone values: `FLT_MODE = { lp: 0.1, hp: 0.3, bp: 0.5, peak: 0.7, notch: 0.9 }`.

**Raum**:

| CC | Macro | CC | Macro |
|---|---|---|---|
| 20 | Feedback | 26 | Reverb |
| 21 | Predelay | 27 | Modulation |
| 22 | Mix | 28 | Rate |
| 23 | Size | 30 | Diffusion |
| 24 | Low Cut | 31 | Decay |
| 25 | High Cut | 14 | Damp |
| 16 | Freeze (always 0) | | |

Mode (15) and Density (29) are stepped, and their zones haven't been measured yet. Don't automate them until I measure them.

## The design (copy from `sot 0.96`)

- **Every mode sets every CC.** A live mode switch must never keep the last genre's values. Three layers merge:
  - `FX_BASE` — every CC, for all modes.
  - `FX_GENRE[mode]` — per-genre ranges.
  - `FX_MODES[mode]` — hand-built signals, such as drop-following automation.

  Spec format: a number means static. `[lo, hi]` means breathe and evolve. `{ hold: [lo, hi] }` means one value per song.
- **Breathing and evolving** (`fxEvolve`) — each song picks a center in the middle half of the range, and a slow perlin wanders ±a quarter of the range around it. The SEED sets its rate and phase, so each seed has its own identity. The value never leaves `[lo, hi]`.
- **Held per song** (`fxHold`) — for knobs that warp or jump when moved, like delay times.
- **Stepped** (`fxPick`) — one weighted choice per song, like Filter Mode.
- **Coupled LFOs** (`breathIn`) — squeezes a shared LFO into a per-song window, so coupled knobs keep moving together.
- **Glide** — on every re-evaluate, each CC slides from its last sent value to the new one over `GLIDE_BARS` (8). It uses `globalThis.__sotFx`. Filter Mode and Freeze skip the glide.
- **Clamp** every value to 0–1 before sending.
- Sends for voices a mode doesn't play are 0.
- Drop-aware signals exist: `DROP_RAMP_SIG`, `DROP_ACTIVE_SIG`, `DROP_BOOST_SIG` and `DROP_SWELL_SIG`, which peaks on a drop's first bar and decays through it. The siblings' drop logic may differ, so check it before reusing these.

## Lessons from building SoT (don't repeat these)

- **A `//` comment placed mid-line swallowed `.midichan().midi()`**, and nothing was sent. Keep comments at line ends, after the full expression.
- **Multiplying a signal by 0 to "switch it off" still sends a 0**, and pinned my knobs. Only send what a mode lists.
- **Don't trust CC or knob names in old code.** I remapped several knobs. Ask me for a screenshot of the rack.
- **Stepped knobs need measured zones.** My first guess at Filter Mode had LP/HP and Peak/Notch swapped.
- **Test before every push.** Evaluate the file in Node with `@strudel/core@1.2.5`, `@strudel/mini@1.2.5` and `@strudel/transpiler@1.2.5`. Version 1.2.6 has a broken import. Stub `Pattern.prototype.midi` to collect the patterns, query about 300 cycles, and check every `ccv` is a number in 0–1 and the CC list per mode is what you expect. Say clearly that this proves the code, not the sound.

## Where to start

1. Read `sot 0.96`'s FX section and the sibling's engine: its voices, channels, modes and drop logic.
2. Tell me in a few lines what the sibling has, and what differs from SoT (channels, voices, drop logic).
3. Propose FX character per genre, one genre at a time, with each value labelled fact, logic or guess. Research real production practice, and say how strong the sources are. My starting hunches, all guesses:
   - **DnB**: bass dry, tight delays on the snare, a short bright room.
   - **Jungle**: breaks into reverb, dub-style throws.
   - **Dub**: a heavy A→B chain. My dub build had Raum in series after Replika.
   - **Dubstep**: sub fully dry, snare into a big reverb, half-time delays.
4. Build one genre, test it in Node, push it, and wait for me to confirm by ear before the next one.
