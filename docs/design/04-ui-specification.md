# UI Specification — Verbaria: Wordbound Rising

Portrait, mobile-first, touch-friendly. Visual tone: clean, colorful,
readable at small sizes, never visually noisy — English content must read
as native game UI, not an embedded school-app widget. A single consistent
frame (parchment-and-rune-glass motif, original art direction) wraps both
combat screens and English-challenge screens so the transition between
"fighting" and "answering" never feels like leaving the game.

## Navigation shell

Bottom tab bar, 5 primary destinations always visible; the remaining 4
screens (Quests, League, Guild, Profile) are reached from a secondary menu
button to avoid overcrowding a child-friendly touch target layout:

```
[ HOME ] [ BATTLE ] [ ENGLISH ] [ EQUIPMENT ] [ MORE ▸ ]
                                                  ├─ Quests
                                                  ├─ League
                                                  ├─ Guild
                                                  └─ Profile
```

## 1. HOME

- Hero portrait + current world backdrop.
- At-a-glance: Character Level, Combat Power, Gold, Runeshards, current
  world name and stage.
- Primary CTA button: "Continue Battle" (deep-links to Battle screen at
  last stage).
- Secondary row: today's daily-quest progress ring (e.g., "3/5 daily
  goals"), tapping opens Quests.
- Small non-intrusive banner slot for events/season countdown.

## 2. BATTLE

- Top: monster HP bar + player HP/Focus bars.
- Center: combat viewport (auto-battle animation).
- English Prompt Card slides up from the bottom at scripted intervals —
  full-width, large tap targets (≥48dp), 4-choice grid or single text
  input depending on `question_type`. A visible timer ring (generous,
  untimed-feeling but present) avoids anxiety while still creating pace.
- On correct answer: card flashes success color, brief damage-multiplier
  callout ("+300% Wyrm Strike!") plays over the combat viewport so the
  reward is seen landing on the monster, not just as a stat popup.
- On incorrect: card shows the correct answer + one-line explanation +
  "Added to your Weak Words" chip, then dismisses — no fail sound harsh
  enough to feel punishing.
- Focus meter fills from correct answers; tapping the glowing special-
  attack button once full triggers the manual special attack.
- Post-stage summary sheet: XP/Gold/loot gained, one line of English
  progress ("Vocabulary +40 XP").

## 3. ENGLISH (hub, not a separate app)

Framed as "The Athenaeum" in-fiction — a hero's training ground, styled
consistently with the rest of the game chrome.

- Five large skill tiles (Vocabulary, Grammar, Reading, Listening,
  Speaking), each showing current level + a progress ring + a rotating
  "next reward" preview (an item icon whose special effect keys off that
  skill, tying directly back to §7 of the GDD).
- Tapping a tile opens that skill's practice flow (question card UI shares
  the same visual language as the in-battle prompt card, so there is one
  question-card component reused everywhere, not two competing UIs).
- A "Weak Words" tile surfaces the Revenge Battle entry point (§22 GDD)
  with a small flame/urgency icon on due-for-review items.
- Listening tile includes a visible speaker/replay button; Speaking tile
  shows a big mic button with a waveform animation while recording and the
  four-axis score readout (Pronunciation/Accuracy/Fluency/Overall) after.

## 4. EQUIPMENT

- Paper-doll hero silhouette with 7 slots (Weapon, Armor, Helmet, Boots,
  2x Accessory, plus a Rune-socket summary badge on equipped gear).
- Tapping a slot opens a filtered Inventory list for that slot, sorted by
  rarity/power by default.
- Selected item detail panel shows base stats, rarity, level, socketed
  runes, and its English-linked special effect spelled out in plain
  language ("Answer 5 vocabulary questions correctly in a row to trigger
  Wyrm Strike (+300% damage)") — the English requirement is always visible
  here, not hidden in a tooltip.
- Upgrade button shows Gold/Runeshard cost and a preview of the post-
  upgrade stat line.

## 5. INVENTORY

- Grid view, filter chips (slot, rarity, "socketed" toggle), sort menu
  (power, rarity, newest).
- Bulk "salvage" flow for low-rarity duplicates into Runeshards (with a
  confirmation step — irreversible action).

## 6. QUESTS

- Two tabs: Daily / Weekly.
- Each quest card shows a paired objective line (combat + English) and a
  single reward icon, mirroring the GDD's paired-quest design so the UI
  never separates "the RPG part" from "the English part" of a quest.
- Progress bars per quest; claim button enabled only on completion.

## 7. LEAGUE

- Season banner (countdown timer, current tier badge: Beginner/
  Intermediate/Advanced).
- Mode selector: Weekly League, Boss Race, Vocabulary Challenge, Grammar
  Arena, Listening Battle, Survival Challenge — each a card with a short
  rule line and a "Play" button.
- Leaderboard list: rank, player name/avatar (moderated, no PII), score,
  tier badge. Score breakdown expandable per row (accuracy, difficulty
  bonus, streak, etc.) so scoring never feels opaque.
- Rewards preview strip at the top showing chest tiers by rank band.

## 8. GUILD

- Guild header: name, castle tier, total Guild XP progress bar to next
  tier.
- Member list with each member's XP contribution this week (mirrors the
  GDD example table).
- Social interaction restricted to a preset-phrase/emote picker (no open
  text field) — this is a hard UI constraint, not just a backend filter,
  so there is no code path that renders a free-text chat bubble.
- Guild Chest claim button, Guild Castle upgrade screen (spend pooled
  materials, cosmetic-only structural upgrades + passive bonuses).

## 9. PROFILE

- Header: avatar, Player Level, Combat Power, Win Rate, Weekly Rank.
- Five skill-level bars (Vocabulary/Grammar/Reading/Listening/Speaking)
  matching the GDD §4 mock.
- Equipment Power summary, Boss Progress checklist (per world).
- "Weak Words," "Words Mastered," and "Recent Mistakes" panels — framed
  positively (mastery counts are the hero number, weak words are a to-do,
  never a shame list).

## 10. PARENT DASHBOARD (separate entry point, PIN-gated)

Not part of the child-facing tab bar — reached via a small, unobtrusive
"Parents" link on the app's title/settings screen, behind a simple PIN or
system-auth gate.

- Weekly study time chart.
- Questions answered / accuracy, overall and per skill (bar chart).
- Per-skill progress lines (Vocabulary/Grammar/Reading/Listening/Speaking)
  over the last 4–8 weeks.
- Weak Words and Words Mastered lists (same underlying data as Profile,
  presented calmly — no rank, no comparison to other children).
- Settings toggles: social features on/off, purchases on/off, daily
  reminder on/off.

## Shared components (built once, reused everywhere)

- `QuestionCard` (used in Battle, English hub, League challenge modes).
- `RewardPopup` (XP/Gold/loot, used after any completed activity).
- `SkillProgressBar` (used in English hub and Profile).
- `ItemDetailPanel` (used in Equipment and Inventory).
- `ChestOpenAnimation` (used in Quests, League, Guild, Daily/Weekly claims).

Building these as shared components early (Phase 3–5) avoids the "quiz app
bolted onto an RPG" look the brief explicitly warns against — every screen
that shows an English question or a reward uses the same visual system as
combat.
