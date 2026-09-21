# Smallest Playable Prototype

Implements the pre-Phase-3 validation slice defined in
[`../docs/design/05-roadmap-and-risks.md`](../docs/design/05-roadmap-and-risks.md#1-smallest-playable-prototype-validate-the-core-loop-first).

`whisperwood-vocab-battle.html` is a single self-contained HTML/CSS/JS
Canvas build (no dependencies) that runs in any browser — used here instead
of a Unity build because Unity isn't available in this execution
environment, and the only thing this step needs to validate is **whether
the loop is fun**, not production code.

## v2: real-time movement (revised from v1's static auto-battler)

The first version was a static, fully automatic battle (no player
movement) and read as too passive/low-agency. v2 replaces that with a
free-movement top-down arena, closer to the action-idle feel of games like
Archero/Brotato/X-HERO — movement is manual, combat is automatic:

- **Virtual joystick** (bottom-left) moves the Wordbound freely around the
  arena — used to dodge Hushspawn contact damage or reposition.
- **Auto-aimed ranged attacks**: the hero automatically fires at the
  nearest enemy in range on a cooldown — no manual aiming, so the
  joystick's whole job is movement/positioning.
- **Dash Strike** (bottom-right button): once the Focus meter is full, tap
  to dash in the joystick's current direction, dealing damage to any
  Hushspawn along the dash path — a second, player-triggered power lever
  distinct from the streak-triggered special.
- Hero now has real HP and can take contact damage from enemies that reach
  it (with brief invulnerability after a hit) — movement has actual
  stakes, but per GDD §18 a "defeat" only triggers a soft recovery (partial
  HP refill), never a hard game-over, staying child-friendly.
- 3 waves of Whisperwood Hushspawn (Mumbleling → Gruntling Goblin →
  Underbrush Wolf, mixed in increasing numbers), matching the GDD §8
  roster, drawn on an HTML canvas rather than static sprites.

The English-combat loop itself is unchanged from v1:

- Vocabulary questions only (12-question original bank), surfaced on a
  fixed ~5s cadence of active play time — no grammar/reading/listening/
  speaking yet. The arena pauses while a question is open, so reading time
  is never combat-punished.
- Correct answer → 4s "Empowered" buff (faster, harder auto-attacks),
  Focus gain, Vocabulary XP, streak counter.
- **5 correct answers in a row → "Wyrm Strike"**, an AOE nova around the
  hero — the Wyrmfang-Blade-style item effect from GDD §7, now visibly
  clearing surrounding enemies rather than a single bonus hit.
- Wrong answers: no combat penalty beyond the missed bonus, correct answer
  + example sentence shown immediately, word logged to a session
  Weak-Word list — never punitive, per GDD §18.
- End-of-session summary (accuracy, new words learned, monsters defeated,
  best streak, weak words) framed positively, per GDD §18/§23.
- Still no save system, no inventory, no worlds beyond Whisperwood —
  intentionally out of scope for this step (see roadmap doc).

## How to use it

Open `whisperwood-vocab-battle.html` directly in a browser (or in the
published Artifact — ask in-session for the current link). Drag the
bottom-left stick to move, tap the bottom-right button once Focus is full.
A full 3-wave run takes a few minutes.

## Status

**Built, not yet validated.** Per the roadmap's exit criterion: this
should not move to Phase 3 (real Unity combat prototype) until playtesters
report the vocabulary prompts feel like *part of the fight* rather than an
interruption, and specifically that landing the 5-streak Wyrm Strike feels
like an earned combat moment. Record playtest feedback here (or in a linked
doc) once gathered.
