# Smallest Playable Prototype

Implements the pre-Phase-3 validation slice defined in
[`../docs/design/05-roadmap-and-risks.md`](../docs/design/05-roadmap-and-risks.md#1-smallest-playable-prototype-validate-the-core-loop-first).

`whisperwood-vocab-battle.html` is a single self-contained HTML/CSS/JS
Canvas build (no dependencies) that runs in any browser — used here instead
of a Unity build because Unity isn't available in this execution
environment, and the only thing this step needs to validate is **whether
the loop is fun**, not production code.

## v4: combat-triggered quizzes, town gate route, power-gating, minimap, 3 worlds, race-specific weapons

- **Vocabulary quizzes now only fire during an actual encounter.** The
  prompt timer only counts up while a monster has aggro'd the hero
  (`currentEngagement()`), and the difficulty tier is taken from that
  specific engaged monster's level — not a global timer independent of
  combat. Wandering with no monster nearby never interrupts you.
- **A real route back to town**: a Town Gate is drawn as a landmark in the
  world (near the hero's spawn point); walking into it triggers the return
  to town automatically, in addition to the instant "🏠 Town" button. The
  world is also divided into distance-from-gate bands (edge → heart →
  depths), each raising the base monster level, so the map now has a
  legible sense of "further in is more dangerous."
- **Power-gated monsters**: a monster whose level exceeds
  `player.level + weaponTier + armorTier + bootsTier + 1` is shown greyed
  out with a 🔒 and never aggros, attacks, or takes damage — directly
  implementing "you need to level up or gear up before you can fight
  stronger monsters." Leveling up or buying gear unlocks them live, mid-run.
- **Minimap** (top-right of the arena) shows the gate, every monster
  color-coded by quiz difficulty (grey if power-locked), the hero, and the
  current camera viewport — since the world is now several screens wide.
- **Three selectable worlds** from the Town hub's "Choose Your
  Battleground" panel: Whisperwood Forest (Lv.1+), Glimmerglass Dunes
  desert (Lv.4+), and Tideglass Coast (Lv.7+) — original names/creatures
  per biome (e.g. desert's Duneworm/Mirage Wraith/Sandfang Jackal, coast's
  Brinewisp/Tideclaw Crab/Kelpfang Serpent), sharing the same combat/quiz
  mechanics but with distinct background art, palette, and decorative
  props (dunes/cacti vs. rocks/kelp vs. trees/grass).
- **Race-specific weapons and silhouette details**: Elf carries a bow,
  Dwarf a stubby warhammer, Orc Warrior a wide cleaver plus war-paint and
  spiked pauldrons, Human a straight sword and a small back shield —
  instead of every race swinging an identical blade.

## v3: login, race select, open-world forest, leveling & shop

Adds a full session flow around the combat loop: a name-entry "login"
screen, a character-select screen with 4 original playable races (Human,
Elf, Dwarf, Orc Warrior — original stat spreads and art, not tied to any
specific existing game or IP), and an open-world Whisperwood the hero
roams with the joystick (camera follows the hero across a map several
screens wide, instead of a single fixed arena).

- Monsters are no longer fixed waves: they're scattered through the
  forest at a level (1–8, drifting with the player's own level) shown as
  a badge over their head, color-coded by the vocabulary tier they'll
  quiz on approach (green/gold/red = easy/medium/hard) — directly
  implementing "word difficulty should scale with monster level" from the
  GDD's adaptive-difficulty principle (§32).
- Monsters only aggro (chase/attack) once the hero gets close, so the
  player chooses which fights to pick rather than being swarmed.
- Defeating a monster grants XP and gold; enough XP levels the character
  up (more max HP/attack, per GDD §3's character-level track); a
  defeated monster respawns elsewhere after a few seconds so the forest
  stays populated.
- A "🏠 Town" button returns to a hub screen at any time — shows
  level/gold/words-learned, and a Blacksmith & Outfitter shop where gold
  buys Weapon/Armor/Boots upgrade tiers (small, GDD-§11-style item system:
  each tier is a flat stat bump, gated by cost and by owning the previous
  tier). Purchases apply the next time you re-enter the forest.
- Character race sets starting HP/Speed/ATK (Human balanced, Elf fast but
  fragile, Dwarf slow but tanky, Orc Warrior hard-hitting but slower) —
  all original silhouettes/palettes, not copies of any specific existing
  game's character designs.

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
