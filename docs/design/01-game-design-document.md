# Game Design Document — Verbaria: Wordbound Rising

## 1. Fiction & Setting (original IP)

The world of **Verbaria** is being consumed by **the Hush** — a spreading
corruption that garbles language, drains meaning from words, and curdles
broken sentences into monsters ("**Hushspawn**"). Every Hushspawn is a
grammar mistake, a mispronounced word, a forgotten meaning given claws.

The player is a **Wordbound**: a hero whose life-force and combat power are
literally bound to language clarity. This is the in-fiction justification for
the core mechanic — English mastery is not a mini-game bolted onto combat,
it is the source of the hero's power within the story itself. Restoring a
word's meaning restores a fragment of Verbaria.

All names below (worlds, monsters, items, currencies) are original to this
project.

- **Gold** — common currency, dropped by monsters, used for basic upgrades.
- **Runeshards** — crafting/upgrade material, dropped rarely, used for
  higher-rarity upgrades and rune socketing.
- **Focus (FP)** — Battle Energy. Spent to trigger special attacks and
  skills; regenerated primarily by answering English prompts correctly
  during combat.
- **League Points (LP)** — competitive ranking currency, earned through the
  Weekly League and other competitive modes (see §16).

## 2. Core Gameplay Loop

```
ENGLISH LEARNING (vocab / grammar / reading / listening / speaking)
        ↓
XP · Gold · Focus (battle energy) · League Points
        ↓
ITEMS (weapons, armor, runes, materials)
        ↓
STRONGER WORDBOUND (character level, equipment power, English level)
        ↓
STRONGER HUSHSPAWN (harder worlds unlock)
        ↓
BETTER LOOT
        ↓
HARDER WORLDS → HARDER ENGLISH CONTENT
        ↓
COMPETITION (leagues, boss races, arenas)
        ↓
BETTER REWARDS (primarily items, not stat shortcuts)
        ↓
REPEAT
```

English is never a side quiz. Concretely, this means:

- Combat itself pauses for an English prompt at fixed, predictable
  intervals (not random interruptions), and the prompt's *outcome directly
  computes* the next combat action's magnitude (damage multiplier, crit
  chance, energy gained) rather than merely "unlocking" a separate reward
  screen.
- Idle/auto-combat continues at a baseline rate without English input, so
  the game is still an idle RPG at its core — but English performance is
  the single largest lever on power, output, and unlock speed. A player who
  never engages with English still has a game; a player who engages well
  clearly outperforms them.
- Item special effects (§12) key directly off English performance streaks,
  so "using an item well" and "practicing English well" become the same
  verb.

### Moment-to-moment combat loop

```
MONSTER SPAWNS
   → auto-attack ticks (baseline DPS from gear/level)
   → every N seconds / every M kills: English Action Prompt appears
        → CORRECT → damage multiplier + Focus gain + skill charge + skill XP
        → INCORRECT → reduced multiplier, word/rule logged to Weak-Word Ledger,
          brief in-context explanation shown (no combat penalty beyond the
          missed bonus — never punitive to a child player)
   → Focus threshold reached → player-triggered special attack
   → MONSTER DEFEATED → XP + Gold + loot roll
   → NEXT MONSTER
```

## 3. Player Progression System

Three progression tracks run in parallel and gate each other:

1. **Character Level** — from cumulative XP (combat + English combined).
   Raises base stats and unlocks skill slots.
2. **Equipment Power** — from items equipped (see §11). Raises damage/HP/etc.
   directly.
3. **English Skill Levels** — five independent tracks (§4). Each gates
   specific world content, item special effects, and league scoring.

World access requires **both** a minimum Equipment/Character Power **and**
minimum English skill levels (§14) — this is the deliberate anti-"pay past
the English" design choice: gold and grinding alone cannot open late-game
content.

## 4. English Skill System

Five independent skill tracks, each with its own level curve (1–60 in the
MVP content plan, extensible):

| Skill | Primary combat effect |
|---|---|
| Vocabulary | Base damage bonus, unlocks item vocabulary-effects |
| Grammar | Skill Power (ability damage) bonus, unlocks grammar-gated worlds |
| Reading | XP gain multiplier, unlocks lore/quest content |
| Listening | Crit chance, unlocks special-attack charge rate |
| Speaking | Special/ultimate attack unlock, unlocks Speaking-gated worlds |

Each skill levels from its own XP pool, earned only through that skill's
question type, so a player cannot "grind vocabulary" to raise Speaking.
Character Level uses a separate pooled XP that is a fraction of all five.

Player-facing summary (Profile screen):

```
Vocabulary  Lv. 25   ▓▓▓▓▓▓▓▓░░  62%
Grammar     Lv. 20   ▓▓▓▓▓░░░░░  48%
Reading     Lv. 23   ▓▓▓▓▓▓▓░░░  55%
Listening   Lv. 18   ▓▓▓▓░░░░░░  30%
Speaking    Lv. 16   ▓▓▓░░░░░░░  20%
```

### 4.1 Vocabulary

Question types: multiple choice meaning, synonym, antonym, fill-in-the-blank,
spelling, word matching, context-based usage.

```
"What does 'reluctant' mean?"
A. unwilling   B. excited   C. angry   D. careless
```

- Correct → XP, Gold, Focus, Vocabulary XP, chance of a bonus Critical
  Attack window on the next auto-attack.
- Incorrect → the correct meaning + one example sentence is shown
  immediately (never just "wrong"); the word is added to the **Weak-Word
  Ledger** (§22) with a spaced-repetition schedule.

### 4.2 Grammar

Categories: subject-verb agreement, verb tense, articles, prepositions,
pronouns, modals, conditionals, passive voice, comparatives, relative
clauses, sentence correction, sentence completion. Difficulty ramps by
CEFR band (see §25 metadata) rather than raw question count, so two
players at different levels see different grammar content even in the
same world.

### 4.3 Reading

Short passages (length and lexical difficulty scaled to player CEFR
level), with question types: main idea, detail, inference, vocabulary in
context, reference, sequence, author's purpose. Completing a passage's
question set grants Reading XP plus a flat Gold/loot bonus (reading takes
longer than a single vocab question, so its reward is bundled per-passage,
not per-question, to keep reward rate roughly comparable across skills).

### 4.4 Listening

Uses real audio via the platform TTS engine at launch, with a path to
licensed native-voice recordings post-launch (see §30 in architecture doc).
Never text-only.

```
🔊 "The boy forgot to bring his umbrella."
"What did the boy forget?"
A. Backpack   B. Umbrella   C. Phone   D. Shoes
```

Correct answers can increase crit chance, charge special attacks, or add
flat battle damage; results also add Listening XP.

Progressive difficulty ladder:

1. Single words
2. Short sentences
3. Short two-line exchanges
4. Longer multi-turn conversations
5. Short natural-speed stories/dialogues

### 4.5 Speaking

Uses the device microphone via an abstracted `ISpeechRecognitionService`
(architecture doc §5). The game presents a situational prompt from an NPC
and asks for a spoken response evaluated on four independent axes, never on
exact string matching:

- **Pronunciation** (phoneme-level confidence score)
- **Sentence accuracy** (semantic/structural match to acceptable answer set)
- **Fluency** (pace, hesitation, filler-word rate)
- **Target-word / contextual appropriateness** (did the response use the
  required vocabulary/grammar point and fit the situation)

```
NPC: "Where are you going?"
Player (spoken): "I'm going to school."
```

Each accepted target has a small set of paraphrase-tolerant acceptable
forms (see Data Model §"SpeakingPrompt" — `accepted_patterns[]`), e.g.
target "I am going to school." accepts "I'm going to school", "I'm going to
school now", etc. Matching uses intent/slot matching (key verb + destination
present, tense correct) rather than exact text.

```
Pronunciation: 91
Accuracy:      95
Fluency:       87
Overall:       91
```

High Overall scores can trigger special attacks unique to Speaking-keyed
items (§12).

### 4.6 English as a Control Mechanism (optional advanced mode)

An optional mode, off by default and never required for main-line
progression, where spoken short commands drive the hero directly:
"Attack the dragon!", "Use the shield!", "Run!", "Heal me!" mapped via
intent classification (not literal string match) to existing combat
actions. Framed in-game as a late-game "Voice Bond" ability unlocked after
reaching a minimum Speaking level, so it never gates early progress and
never becomes mandatory.

## 5. Combat System

- **Auto-battler core**: the Wordbound auto-attacks continuously; the
  player's job is item/skill loadout plus timed English prompts and manual
  special-attack activation once Focus is full.
- **Depth without complexity**: 4 equipped active skills (from Skill
  Books, see §11), each with a cooldown and a Focus cost; skill *damage*
  scales with Grammar level, skill *unlock* for ultimates scales with
  Speaking level, *crit* scales with Listening level, base damage scales
  with Vocabulary level, and XP-from-combat scales with Reading level. This
  is the concrete "English affects performance" wiring requested by the
  brief — every one of the five skills maps to a distinct, legible combat
  stat rather than all five vaguely boosting "power."
- Combat difficulty inside a world scales with a stage counter; bosses gate
  progression to the next stage and always include an English-gated
  mechanic (e.g., a boss "shield" that only breaks on a correct grammar
  answer).

## 6. Item System

Slots: Weapon, Armor (chest), Helmet, Boots, Accessory (x2), Rune (x3
sockets, socketed into weapon/armor).

Rarity tiers: **Common → Uncommon → Rare → Epic → Legendary → Mythic**,
each with a stat multiplier band and a distinct visual tier (color +
silhouette complexity), no rarity-specific proprietary iconography.

Example weapon ladder (original names/numbers, illustrative only — final
numbers belong in the balancing pass, §"Balancing" in roadmap doc):

| Item | Rarity | ATK |
|---|---|---|
| Oakwood Blade | Common | +10 |
| Ironclad Blade | Uncommon | +35 |
| Bastionguard Sword | Rare | +80 |
| Wyrmfang Blade | Epic | +180 |
| Verbaria's Edge | Legendary | +400 |

Every item carries: base stats, level requirement, rarity, an optional
**English-linked special effect** (§7), an upgrade cost curve (Gold +
Runeshards, scaling per level), and a socket count.

## 7. English-Linked Item Abilities

This is the mechanical heart of "English *is* the RPG system," not an
add-on. Every rarity tier from Rare upward has at least one English-linked
effect; lower tiers can have simpler ones. Examples (original names):

| Item | Effect condition | Bonus |
|---|---|---|
| Wyrmfang Blade | 5 correct vocabulary answers in a row | "Wyrm Strike": next attack +300% damage |
| Sagewood Staff | Grammar accuracy ≥ 90% this session | Skill Power +25% |
| Echoing Longbow | 5 consecutive correct listening answers | Guaranteed critical hit |
| Tome of Clarity | Complete a reading challenge | XP +20% for 5 minutes |
| Voxbound Charm | Speaking Overall score ≥ 85 | Unleash a bonus special attack |
| Ledger of Old Words | Correctly answer a Weak-Word Ledger review | Bonus Gold +50% for that kill |

Design rule: every new item template must declare which English skill (or
skills) its special effect keys off, so content authors cannot accidentally
ship items disconnected from the learning loop.

## 8. Monster System (Hushspawn)

Monsters are always framed as corrupted language — this keeps flavor
consistent and 100% original. Example roster by world (see §9 for full
world list):

- **Whisperwood** (beginner): Mumbleling, Gruntling Goblin, Underbrush
  Wolf
- **Duskmire Thicket** (dark forest): Snarlbrute Orc, Nightfang Wolf,
  Hexmutter Witch
- **Bastion of Broken Verbs** (demon-castle analog): Bonemouth Knight,
  Warpling Demon, Inkshade Mage
- **Wyrmreach** (dragon world): Cindertongue Wyrm, Elder Wyrm, Wyrm
  Sovereign

Each monster has: HP, Attack, Defense, Speed, Element (Verdant / Umbral /
Ember / Runic / Aether — original elemental set, 5 types with a simple
weakness triangle plus one neutral), a defined Weakness, and a Loot Table
(gold range, drop-rate-weighted item pool, guaranteed-vs-chance rune drop).

## 9. World Structure

| World | Theme | English focus |
|---|---|---|
| 1. Whisperwood | Beginner forest | Basic vocabulary |
| 2. Bastion of Broken Verbs | Grammar castle | Grammar |
| 3. Glimmerglass Dunes | Vocabulary desert | Vocabulary |
| 4. The Athenaeum Spire | Reading kingdom/library | Reading |
| 5. Emberlisten Caldera | Listening volcano | Listening |
| 6. Voxhaven | Speaking city | Speaking |
| 7. Wyrmreach | Dragon kingdom | Mixed, advanced (all 5 skills) |

Both combat difficulty and English difficulty rise together per world;
World 7 deliberately mixes all five skills to serve as the "capstone"
content and primary League/Boss-Race venue.

## 10. Progression Gating

Raw ATK is a necessary but not sufficient gate. Example combined gates:

| World | Min. Equipment Power | Additional requirement |
|---|---|---|
| Whisperwood | 50 ATK | — |
| Duskmire Thicket | 150 ATK | Vocabulary Lv. 10 |
| Bastion of Broken Verbs | 400 ATK | Grammar Lv. 15, prior boss cleared |
| Wyrmreach | 1000 ATK | Reading Lv. 20 + Listening Lv. 20 + Speaking Lv. 15 |

This is the deliberate mechanism preventing "buy gear, skip English":
whales can shortcut Equipment Power but cannot shortcut skill-level gates,
because skill levels only rise from correctly-answered, skill-tagged
questions.

## 11. Competitive System

Modes:

1. **Weekly League** — primary ladder, seasonal (§13).
2. **Boss Race** — first-to-clear a rotating boss under a shared seed.
3. **Vocabulary Challenge** — timed vocabulary-only gauntlet.
4. **Grammar Arena** — timed grammar-only gauntlet, 1v1 asynchronous score
   comparison.
5. **Listening Battle** — timed listening-only gauntlet.
6. **Survival Challenge** — endless combat wave run scored by depth reached,
   with periodic mandatory English checkpoints that gate wave advancement.

### Scoring model (Weekly League, illustrative weights)

```
Score = (Accuracy_weighted × 40)
      + (Difficulty_bonus × 25)          // harder questions worth more
      + (NewWordsLearned × 10)
      + (ConsecutiveCorrectStreak × 10)
      + (CombatProgressDelta × 15)
```

Explicitly **not** "total questions answered," so grinding easy questions
cannot outscore a player answering fewer, harder, more-accurate questions.
`Difficulty_bonus` is derived from each question's `difficulty`/`CEFR_level`
metadata field (§Data Model).

## 12. Competitive Rewards

Primarily items and item-adjacent resources, not raw stat boosts or
pay-only exclusives:

| Rank | Reward |
|---|---|
| 1–3 | Legendary Chest |
| 4–10 | Epic Chest |
| 11–30 | Rare Chest |
| Everyone who completes ≥1 league match | Basic participation reward |

Non-paying players can reach top chests purely through play; the
Battle Pass (§17) only offers cosmetic/convenience track items, never
league-exclusive power items.

## 13. Anti-Snowball System

- **Seasonal leagues** (roughly monthly) with **seasonal equipment** that
  resets/normalizes at season start.
- **Tiered brackets**: Beginner / Intermediate / Advanced, assigned by
  recent progression rate, not lifetime power, so a returning or newer
  player is matched fairly.
- **Promotion/relegation** between tiers each season.
- **Catch-up rewards**: a bonus XP/Gold multiplier for players below the
  season's median progression, decaying as they approach median (never
  handed to top players).
- Permanent single-player progression (character level, non-seasonal
  items) is never reset — only competitive standing and seasonal gear are.

## 14. Guild System

Small guilds (suggested cap: 30 members) where members contribute English
XP to a shared **Guild XP** pool:

```
KNIGHTS OF CLARITY GUILD
  Member A: 2,500 XP
  Member B: 1,900 XP
  Member C: 2,300 XP
  --------------------
  Total: 6,700 XP → unlocks next Guild Castle tier
```

Rewards: Guild Chest, shared materials, cosmetics, Guild Castle
progression (a shared, non-combat structure that unlocks passive
guild-wide bonuses as it's built up through pooled XP).

### Child-safety controls (mandatory, non-negotiable)

- No unrestricted free-text public chat. Guild communication uses a
  **preset phrase/emote system** ("Nice job!", "Let's go!", "GG") plus
  quick-react stickers — no open text field for players under a
  configurable age threshold.
- No mechanism anywhere in guild/social UI that surfaces or requests
  personal information (no real name fields, no external contact fields).
- Parental controls can fully disable social features from the Parent
  Dashboard (§16).
- Report/block available on any player card.
- All social content passes through automated moderation before being
  visible to other players (even preset-phrase selections are logged for
  abuse-pattern detection, e.g. spam-tapping the same phrase).

## 15. Quest System

Quests always pair a combat objective with an English objective:

```
QUEST: Defeat 20 Gruntling Goblins
Requirement: Answer 10 vocabulary questions
Reward: Bastionguard Sword (Rare)
```

```
QUEST: Complete 5 listening challenges
Reward: Echoing Longbow
```

```
QUEST: Speak 10 English sentences
Reward: Voxbound Rune
```

### Daily / Weekly cadence

**Daily** (kept short, ~10–15 min total, per §18 educational principle):
10 vocabulary questions, 5 grammar questions, 1 listening challenge, 3
speaking challenges, defeat 20 monsters.

**Weekly**: learn 50 new words, complete 3 reading passages, complete 20
listening questions, complete 10 speaking challenges, defeat the weekly
boss.

Rewards are primarily in-game items/materials, not currency-only, to keep
the "learning → gear" loop legible.

## 16. Parent Dashboard

A separate, calm, non-competitive view (own login/PIN, no ads, no
purchase prompts) showing:

- Weekly study time
- Questions answered / accuracy (overall + per skill)
- Vocabulary / Grammar / Reading / Listening / Speaking progress charts
- Weak Words list, Words Mastered count
- Toggle: disable social/chat features, disable in-app purchases, set a
  daily play-time reminder (soft, not a hard lock — framed as encouraging
  a stopping point, not restricting access punitively)

Explicitly no leaderboard-vs-other-children view here — this dashboard is
about *this child's* learning, not comparison, per the brief's "not
overly competitive or stressful" requirement.

## 17. Monetization

Allowed: rewarded video ads (opt-in, never forced mid-lesson), cosmetic
skins/effects, convenience items (e.g., extra inventory slots, cosmetic
mount), a Battle Pass with cosmetic/material rewards only, optional
premium content packs (e.g., an extra passage pack) that are supplementary,
never gating the core five-skill curriculum.

Never allowed: paying to skip or unlock core vocabulary/grammar/reading/
listening/speaking content; paywalling league participation; loot boxes
that contain gameplay-power items behind randomized real-money spend
without a guaranteed non-paying path (pity timer / dupe-to-material
conversion mitigates this); ad content or purchase flows targeted at
children beyond applicable child-directed-advertising rules (e.g., COPPA-
equivalent handling — treat as a legal/compliance checklist item, not a
design nicety).

## 18. Educational Principle (guiding constraint on all systems)

Optimize for **meaningful English practice completed while remaining
engaged**, not for raw session length or DAU/session-count vanity metrics.
Concretely: daily quest volume is capped at a level completable in
10–15 minutes; no energy-refill-via-ad loop that encourages marathon
sessions; a gentle end-of-session summary screen ("Great session! You
learned 4 new words today.") functions as a natural stopping point rather
than a re-engagement hook.

## 19. Originality / Legal Compliance Notes

- No character names, monster names, item names, world names, UI
  layout, map layout, music, or code are copied from any existing
  commercial game. All names in this document are newly created for this
  project.
- Generic fantasy archetypes (slime, goblin, wolf, orc, skeleton, dragon)
  are public-domain-of-genre tropes reflavored with original names, stats,
  and an original unifying fiction (the Hush / Hushspawn) — the
  implementation must always pair a generic archetype with the original
  name and an original art pass; no visual or audio assets may be copied
  or "asset-flipped" from another title.
- Any third-party asset (font, SFX pack, TTS/STT SDK) used in production
  must carry a commercial-use-compatible license; track licenses in a
  `THIRD_PARTY_NOTICES.md` once assets are selected (Phase 12 concern,
  flagged in the roadmap doc's risk list).
