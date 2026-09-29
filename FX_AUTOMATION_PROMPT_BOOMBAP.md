# Prompt: FX automation for a boom bap engine

Copy everything below the line into a new Claude Code session on this repo. Don't paste the whole file into chat; tell the session to read it from the repo.

---

You're working in the `hashishmoto/Ship-of-Theseus-` repo. Pull the latest `main` first.

**Your job:** give my boom bap hip hop engine the same FX automation that `sot 0.96` has.

**Read first,** in this order:
1. `FX_AUTOMATION_PROMPT.md`. It holds everything shared: how I work, the rig (Replika on return A, Raum on return B, the one-way A→B send on CC 95, FX on MIDI channel 16), the full CC map with my macro names, the measured Filter Mode zones, the design to copy (`FX_BASE` / `FX_GENRE` / `FX_MODES`, `fxEvolve`, `fxHold`, `fxPick`, `breathIn`, the glide, the 0–1 clamp), the lessons learned, and how to test in Node. Follow all of it. Where it talks about DnB, dub or dubstep, read "boom bap" instead.
2. The FX section at the bottom of `sot 0.96`, from `const MIDI_CHAN_FX` to the end. It's the reference implementation.

**The engine.** I don't know yet which file the boom bap engine is in, or whether it exists. Ask me before anything else. If it doesn't exist, stop. Building the engine itself is a separate job, and we'd plan it together first.

Once you have the engine:
- Tell me in a few lines what it has: voices, MIDI channels, modes, tempo, and how it structures a song (sections, breaks, drops or something else).
- Compare it to SoT, especially the channels. SoT uses perc 1, bass 2, stab/chop 3, lead 4, pad 5. Ask before moving any channel, because I have to re-route tracks in Ableton.

## What's different about boom bap

- **No drops.** The SoT drop signals (`DROP_SWELL_SIG` and the others) probably don't fit. Find what the engine uses to mark structure: verse/hook changes, breaks, a beat switch. Automate against that instead, and ask me if it's unclear.
- **Slower tempo, roughly 85–95 BPM.** `GLIDE_BARS` and `SECS_PER_BAR` then mean longer times, so check the glide still feels right.
- **The sound is dusty and lo-fi,** not washy. Saturation, Wow & Flutter and darker filters probably matter more than reverb size.

## My starting hunches (all guesses, research them)

- **Drums** mostly dry, with a short small room on the snare. The kick and bass stay dry.
- **Sample chops and keys** (stab and pad) go into a tape-style delay and a small dark reverb.
- **Replika:** higher Saturation and Wow & Flutter, a darker HiCut for the vinyl feel, a low-cut on the echoes, and fairly low feedback.
- **A→B** low. Boom bap keeps delay and reverb separate and clean.
- **Delay times** held per song, likely quarter or dotted-eighth feel. Ask me to measure which macro values give which divisions, the same way I measured Filter Mode.
- **Maybe more than one mode:** for example classic 90s (dry, dusty), jazzy (keys and pad in more reverb) and dark (lower HiCut, more wow). Propose modes, don't just build them.

Research real boom bap mixing practice (for example Pete Rock, DJ Premier, J Dilla, and modern lo-fi producers). Say how strong each source is, and label every number fact, logic or guess.

## Order

1. Ask which file the engine is in.
2. Summarize the engine and how it differs from SoT.
3. Propose the modes and the FX character for each. Short, one mode at a time, and wait for my answer.
4. Build one mode. Test it in Node: all CCs valid 0–1 on channel 16, the full CC set in every mode, and the glide working. Push it, give me the raw GitHub link, then wait for me to confirm by ear.
