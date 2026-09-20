# Roadmap, MVP Scope & Risk Assessment — Verbaria: Wordbound Rising

## 1. Smallest playable prototype (validate the core loop first)

Before any of the 12 production phases below, build the **smallest slice**
that can validate whether

```
ENGLISH → REWARD → ITEM → POWER → MONSTER → PROGRESSION
```

is actually fun. Scope for this pre-production prototype (target: a few
days of focused work, throwaway-quality art acceptable):

- 1 hero, auto-attacking a single monster type in one arena.
- Vocabulary questions only (simplest skill to implement — no audio, no
  mic).
- Correct answer → damage multiplier + Focus gain, wired exactly as
  described in GDD §5.
- One weapon with one English-linked special effect (Wyrmfang-Blade-style:
  "5 correct in a row → 3x damage").
- No save system, no inventory UI, no worlds — a single scripted session
  loop that can be played start-to-finish in under 5 minutes.

**Exit criterion**: internal playtesters report the vocabulary questions
feel like *part of the fight*, not an interruption to it, and specifically
that landing "5 in a row" for the special effect feels like an earned
combat moment. If this fails, iterate on prompt cadence/UI before writing
a single line of the full MVP — this is the highest-risk unknown in the
whole project (see Risk #1 below) and the cheapest point to de-risk it.

## 2. MVP scope (Phase 3–5 target, first real release-shaped build)

- 1 Hero, 10–20 monsters, 3 worlds (Whisperwood, Duskmire Thicket, Bastion
  of Broken Verbs — enough to prove the world-gating mechanic).
- 20–30 items across Common–Epic (Legendary/Mythic content can wait —
  the rarity *system* should exist, top tiers don't need to be full yet).
- Vocabulary + Grammar fully implemented; Reading and Listening in basic
  form (short passages / on-device-TTS only sentences); Speaking as a
  prototype (mic capture + simple keyword-slot scoring, not full
  4-axis polish).
- Auto combat, XP, Gold, Equipment, item upgrades.
- One weekly leaderboard prototype (local mock data is acceptable — see
  architecture doc §3.5 `ILeaderboardService`).
- Basic quest rewards, save/load, and the 9-screen UI shell (all screens
  present even if some show "coming soon" placeholders for Guild/League
  advanced modes).

Explicitly out of MVP scope: full 7-world content, Mythic-tier items,
guild system, seasonal leagues/anti-snowball tiering, parent dashboard
(beyond a stub screen), voice-command control mode, AI question-generation
pipeline. These are Phase 6+.

## 3. 12-phase roadmap

| Phase | Deliverable | Gate before moving on |
|---|---|---|
| 1 | Complete Game Design Document (this package) | Stakeholder review/sign-off |
| 2 | Technical architecture + data model (this package) | Architecture review; interfaces agreed |
| 3 | Playable combat prototype (auto-battle, 1 monster, no English yet) | Combat feels responsive at 60fps on a mid-tier Android device |
| 4 | Add Vocabulary + Grammar, wire into combat modifiers | Smallest-playable-prototype exit criterion (§1) met |
| 5 | Inventory + item progression (rarity, upgrades, English-linked effects) | Full loop playable start-to-finish; MVP scope (§2) complete |
| 6 | Reading + Listening (real audio pipeline) | Listening audio latency/quality acceptable on target devices |
| 7 | Speaking (mic capture + scoring service) | Speech evaluation accuracy validated against a human-graded sample set |
| 8 | Competition (Weekly League + at least 2 of the 6 modes) | Scoring model resists "grind easy questions" exploitation in playtesting |
| 9 | Guilds (with child-safety-constrained social features) | Moderation/report/block flow passes a safety review |
| 10 | Parent Dashboard | Usability test with at least one parent |
| 11 | Balance progression & reward economy across all systems | Economy simulation shows no dominant strategy that skips English entirely |
| 12 | Android release prep (store listing, licensing audit, compliance) | Legal/child-safety/advertising compliance checklist cleared |

Test and stabilize after each phase before starting the next, per the
brief's explicit instruction — no phase should begin with an unresolved
regression from the previous one.

## 4. Complexity estimates per subsystem

Relative complexity (S/M/L/XL), for planning purposes, not calendar time:

| Subsystem | Complexity | Notes |
|---|---|---|
| Auto-combat core | M | Standard idle-RPG pattern, well understood |
| Vocabulary/Grammar question engine | S–M | Mostly data + simple evaluators |
| Reading | M | Passage difficulty curation is content-heavy, not code-heavy |
| Listening | M–L | Audio pipeline + on-device TTS quality varies by device |
| Speaking | **XL** | Speech recognition, pronunciation scoring, and false-positive/negative tuning are genuinely hard; budget the most schedule risk here |
| Item/Inventory + English-linked effects | M | Needs the shared event-bus decoupling from architecture doc §1 to stay clean |
| World-gating logic | S | Simple rule evaluation once data model exists |
| League scoring/anti-cheat | L | Getting the weighted formula fair (GDD §11) requires playtesting iteration, not just implementation |
| Guild system + child-safe social | M–L | The *feature* is simple; the *safety review* around it is the real cost |
| Parent Dashboard | S–M | Mostly a read-only view over existing analytics data |
| Adaptive difficulty | M | Algorithm is simple (architecture doc §6); tuning thresholds against real player data takes iteration |
| Backend/multiplayer readiness | L (deferred) | Interfaces built now (architecture doc §3.5); actual server is a separate, later project |

## 5. Risks and technical challenges

1. **Speaking evaluation quality (highest risk).** Children's speech
   recognition accuracy varies widely by accent, age, and background
   noise; an overly strict evaluator punishes good-faith attempts and
   breaks trust in the whole "English affects power" promise. Mitigation:
   ship Speaking as a lower-stakes bonus system first (small bonuses, never
   a hard gate) until confidence in the scoring pipeline is established
   with real usage data; keep `ISpeechRecognitionService` swappable so the
   provider can change without a rewrite.
2. **Fun-vs-friction balance in the core loop.** If English prompts feel
   like interruptions rather than combat beats, the central premise of the
   game fails. Mitigation: the standalone smallest-playable-prototype step
   (§1) exists specifically to catch this before real investment.
3. **Economy exploits that let players skip English.** Any gap in
   world-gating (GDD §10) or league scoring (GDD §11) that lets raw
   spending or grinding substitute for skill levels undermines the
   educational mission and the "fair competition" priority. Mitigation:
   Phase 11 explicitly requires an economy simulation pass looking for
   dominant non-English strategies before balance is considered done.
4. **Child-safety/compliance surface area.** Guild social features,
   any leaderboard display name, and monetization to a child-heavy
   audience all carry real legal exposure (child-directed advertising
   rules, data-privacy rules for minors). Mitigation: preset-phrase-only
   social (no free text) built as a hard UI constraint, not a filter;
   compliance checklist gate at Phase 12; parental controls to disable
   social/purchases entirely.
5. **Content authoring throughput.** A five-skill, multi-CEFR-level,
   multi-world curriculum needs a lot of question content. Mitigation:
   the `content_status`/`content_source` fields in the Data Model (§1) are
   designed from day one to support an AI-assisted authoring pipeline with
   a mandatory human-review gate (`draft → pending_review → approved`)
   before anything reaches players — never live AI generation during
   gameplay, per the brief's explicit constraint.
6. **Device fragmentation on Android.** TTS quality, mic quality, and
   performance vary widely across the Android install base. Mitigation:
   define a minimum supported device tier during Phase 3 and test Speaking/
   Listening specifically on low-end reference hardware, not just flagship
   devices.
7. **Session-length metrics vs. educational-principle conflict.** Standard
   mobile-game engagement tactics (energy-refill ads, infinite grind) work
   against the brief's "meaningful practice over playtime" principle.
   Mitigation: daily quest volume capped by design (GDD §18) and treated as
   a product requirement, not a suggestion — no future feature should be
   allowed to quietly reintroduce a marathon-session incentive.

## 6. Recommended implementation order (summary)

1. Smallest playable prototype (§1) — de-risk the core fun question first.
2. Combat core + save system skeleton (Phase 3).
3. Vocabulary + Grammar wired into combat (Phase 4).
4. Items/Inventory with English-linked effects (Phase 5) — this completes
   the MVP loop end-to-end.
5. Reading/Listening (Phase 6), then Speaking (Phase 7) — deliberately
   last among the five skills because it is the highest-complexity,
   highest-risk subsystem (§5 Risk #1); shipping it after the rest of the
   loop is proven means a Speaking slip never blocks validating the core
   game.
6. Competition (Phase 8), then Guild (Phase 9) — competitive systems need
   a stable single-player economy underneath them first.
7. Parent Dashboard (Phase 10) — low technical risk, can slot in whenever
   analytics from Phase 3+ exist.
8. Full economy balancing pass (Phase 11).
9. Release preparation and compliance audit (Phase 12).
