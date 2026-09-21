# Smallest Playable Prototype

Implements the pre-Phase-3 validation slice defined in
[`../docs/design/05-roadmap-and-risks.md`](../docs/design/05-roadmap-and-risks.md#1-smallest-playable-prototype-validate-the-core-loop-first).

`whisperwood-vocab-battle.html` is a single self-contained HTML/CSS/JS build
(no dependencies) that runs in any browser — used here instead of a Unity
build because Unity isn't available in this execution environment, and the
only thing this step needs to validate is **whether the loop is fun**, not
production code.

## What it implements

- 1 hero ("Wordbound") auto-attacking a sequence of 3 Whisperwood Hushspawn
  (Mumbleling → Gruntling Goblin → Underbrush Wolf), matching the GDD §8
  monster roster.
- Vocabulary questions only (12-question original bank), surfaced on a
  fixed ~5s cadence — no grammar/reading/listening/speaking yet.
- Correct answer → +damage multiplier on the next auto-attack, Focus gain,
  Vocabulary XP, streak counter.
- A single item effect, Wyrmfang-Blade-style: **5 correct answers in a row
  triggers "Wyrm Strike"** — a bonus burst hit, exactly as specified in
  GDD §7.
- A manual special attack once the Focus meter fills, separate from the
  streak-triggered Wyrm Strike (validates both the automatic and the
  player-triggered power levers from GDD §5).
- Wrong answers: no combat penalty beyond the missed bonus, correct
  answer + example sentence shown immediately, word logged to a session
  Weak-Word list — never punitive, per GDD §18.
- End-of-session summary (accuracy, new words learned, monsters defeated,
  best streak, weak words) framed positively, per GDD §18/§23.
- No save system, no inventory, no worlds beyond Whisperwood — intentionally
  out of scope for this step (see roadmap doc).

## How to use it

Open `whisperwood-vocab-battle.html` directly in a browser (or in the
published Artifact — ask in-session for the current link). A full run
(3 monsters) takes under 5 minutes.

## Status

**Built, not yet validated.** Per the roadmap's exit criterion: this
should not move to Phase 3 (real Unity combat prototype) until playtesters
report the vocabulary prompts feel like *part of the fight* rather than an
interruption, and specifically that landing the 5-streak Wyrm Strike feels
like an earned combat moment. Record playtest feedback here (or in a linked
doc) once gathered.
