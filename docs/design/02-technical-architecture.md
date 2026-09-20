# Technical Architecture — Verbaria: Wordbound Rising

Target engine: **Unity (2022 LTS or later)**, C#, URP (2D/2.5D mobile
renderer), Android first (min API level chosen for >95% device coverage at
build time), architected so an iOS build is a platform-layer swap, not a
rewrite.

## 1. Guiding architecture principles

- **Data-driven, not hard-coded.** Items, monsters, worlds, skills,
  questions, and rewards are ScriptableObject/JSON-backed data assets, not
  values baked into C# logic. Designers/content authors should be able to
  add a new monster or 50 new vocabulary questions without touching code.
- **Speech is a swappable service**, never a direct SDK dependency inside
  gameplay code — gameplay only talks to an interface (§5).
- **Local-first for the MVP, backend-ready by contract.** The save system
  and league/guild systems are built against a repository interface so a
  local JSON implementation and a future networked implementation are
  interchangeable.
- **English-system code and combat code are decoupled** through an event
  bus: an `EnglishPromptResult` event carries skill, correctness,
  difficulty, and streak data; combat, item-effect, and league-scoring
  systems all subscribe independently. This lets each subsystem evolve
  without the others knowing about it.

## 2. Project folder structure

```
/Assets
  /Scripts
    /Combat            // auto-battle loop, damage formulas, buffs/debuffs
    /Character          // Wordbound stats, leveling, equipment slots
    /Monster             // Monster runtime behavior, AI, spawn tables
    /Weapons             // weapon-specific effect hooks
    /Items               // item runtime model, rarity/upgrade logic
    /Inventory           // bag, equip/unequip, sorting/filtering
    /Skills              // active/passive skill runtime, cooldown, Focus cost
    /English
      /Vocabulary
      /Grammar
      /Reading
      /Listening
      /Speaking
      /WeakWordLedger    // spaced-repetition engine, shared by all 5 skills
      /AdaptiveDifficulty
    /Progression         // XP curves, level-up, world-gate evaluation
    /Quests              // quest definitions, tracking, reward grant
    /League              // scoring, seasons, matchmaking tier assignment
    /Guild               // guild XP pooling, guild castle progression
    /UI                  // screen controllers, one per top-level screen
    /Save                // ISaveRepository + local JSON implementation
    /Audio               // TTS/audio playback abstraction
    /Speech              // ISpeechRecognitionService + provider adapters
    /Analytics           // gameplay + education event tracking
    /Backend             // future network client stubs (auth, sync, league API)
  /Data                  // ScriptableObject assets (items, monsters, worlds, skills)
  /StreamingAssets
    /Questions           // versioned question-bank JSON (see Data Model doc)
    /Audio                // pre-generated / licensed listening audio clips
  /Prefabs
  /Scenes
```

## 3. Core systems

### 3.1 Combat loop

`CombatController` runs a fixed-tick auto-battle loop:
`AutoAttack → DamageResolver → LootRoller → XPGranter`. It exposes a
public `ApplyExternalModifier(CombatModifier)` API that the English system
and item-effect system both call — combat code has zero knowledge of
"vocabulary" or "grammar," it only knows "a modifier arrived." This keeps
Combat/ and English/ fully decoupled and independently testable.

### 3.2 English system

Each of the five sub-modules (`Vocabulary/`, `Grammar/`, etc.) implements a
common `IEnglishChallengeProvider` interface:

```csharp
public interface IEnglishChallengeProvider
{
    EnglishSkill Skill { get; }
    Question GetNextQuestion(PlayerEnglishProfile profile);
    ChallengeResult Evaluate(Question q, PlayerAnswer answer);
}
```

`AdaptiveDifficulty/` wraps a per-skill selector that samples the next
question's difficulty band from the player's rolling accuracy (see §6
below) rather than a fixed level curve.

`WeakWordLedger/` is shared infrastructure: any provider can push a missed
item (`item_id`, skill, timestamp, review_interval) and any provider can
pull "due for review" items — this implements spaced repetition once,
centrally, instead of five separate ad hoc implementations.

### 3.3 Item / Inventory

Items are `ItemDefinition` ScriptableObjects (static data) + a lightweight
runtime `ItemInstance` (level, socketed runes, current upgrade tier). An
`ItemEffectEvaluator` subscribes to the same `EnglishPromptResult` event
stream combat listens to, and independently decides whether an equipped
item's special-effect condition (e.g., "5 correct vocabulary in a row") is
met, then calls the same `ApplyExternalModifier` API combat exposes.

### 3.4 Save system

```csharp
public interface ISaveRepository
{
    Task<PlayerSaveData> Load(string playerId);
    Task Save(PlayerSaveData data);
}
```

MVP ships `LocalJsonSaveRepository` (encrypted-at-rest JSON in
`Application.persistentDataPath`). The interface is intentionally
transport-agnostic so a later `RemoteSaveRepository` (cloud sync, needed
before any real-money league or guild feature ships) is a drop-in
replacement — gameplay code depends only on `ISaveRepository`.

### 3.5 Backend-ready design (future multiplayer/league)

Nothing in the MVP requires a live backend, but the following are
designed against interfaces so a backend can be added without refactoring
gameplay:

- `ILeaderboardService` — `SubmitScore`, `GetRankings`, `GetSeason`.
  MVP implementation returns a local single-player mock leaderboard
  (§Roadmap MVP scope) seeded with no other data, or an emulated local
  ranking, so the League *screen* is fully testable before a server exists.
- `IGuildService` — guild CRUD, XP contribution, chest claims.
- `IAuthService` — anonymous device ID at MVP, upgradeable to real
  account auth later without touching gameplay call sites.

Recommended production backend shape (not built in MVP, documented for
planning): stateless API service (any mainstream stack) + a managed
relational or document store for player profiles/leagues/guilds, and an
object store for versioned question-bank JSON so content can be updated
without an app release.

## 4. Data-driven content (ScriptableObjects)

Content types authored as ScriptableObjects, each with a corresponding
JSON export path for the question bank specifically (questions need to be
updatable server-side without a rebuild — see Data Model doc):

- `ItemDefinition`, `MonsterDefinition`, `WorldDefinition`,
  `SkillDefinition`, `RewardTable`, `QuestDefinition` — Unity
  ScriptableObjects, shipped in the build, edited via custom Inspector
  tooling.
- `Question` bank — JSON in `StreamingAssets/Questions/<skill>/<version>.json`,
  loaded at runtime, versioned so new content can be pushed via a content
  update without a full app-store release (see Data Model doc for schema).

## 5. Speech technology as a modular service

```csharp
public interface ITextToSpeechService
{
    Task<AudioClip> Synthesize(string text, VoiceProfile voice);
}

public interface ISpeechRecognitionService
{
    Task<SpeechAssessment> AssessSpeech(AudioClip recording, SpeechTarget target);
}

public struct SpeechAssessment
{
    public float Pronunciation;   // 0-100
    public float Accuracy;        // 0-100
    public float Fluency;         // 0-100
    public float Overall;         // weighted composite
    public bool TargetWordUsed;
    public bool ContextAppropriate;
}
```

- **Prototype/MVP**: a `MockSpeechRecognitionService` implementation using
  on-device speech-to-text (Android `SpeechRecognizer`) plus a simple
  keyword/slot-matching evaluator against `SpeechTarget.AcceptedPatterns[]`
  — good enough to validate the loop without a paid API dependency.
- **Production**: swap in a real pronunciation-assessment provider behind
  the same interface (evaluated at Phase 7/12, provider chosen based on
  per-call cost, language coverage, and offline fallback needs — an
  explicit open decision, not pre-committed here to avoid locking the
  architecture to one vendor).
- Listening audio: `ITextToSpeechService` backed by on-device TTS for MVP;
  production can mix in licensed native-voice recordings for the
  higher listening difficulty tiers (§4.4 in GDD) via the same interface
  (a `PrerecordedTTSService` that serves clips from
  `StreamingAssets/Audio/` when available, else falls back to on-device
  TTS).

## 6. Adaptive difficulty algorithm

Per skill, maintain a rolling accuracy window (last ~20 answered
questions):

```
if rolling_accuracy(skill) > 0.85:
    next_difficulty_band = current_band + 1   // raise difficulty
elif rolling_accuracy(skill) < 0.55:
    next_difficulty_band = current_band - 1   // ease off, more review items
else:
    next_difficulty_band = current_band       // hold, inject Weak-Word Ledger items
```

Applied **independently per skill** (this directly implements the brief's
example: 95% vocabulary accuracy raises vocabulary difficulty while 65%
grammar accuracy holds grammar steady with extra practice, instead of one
global "character level" difficulty knob). Difficulty band maps to the
`difficulty`/`CEFR_level` fields on the `Question` schema (Data Model doc).

## 7. Analytics

Two independently-tagged event streams (same underlying analytics
pipeline, different event namespaces, so gameplay-tuning and
learning-efficacy analysis never get mixed in dashboards):

- **Gameplay**: `session_start/end`, `monster_defeated`, `item_obtained`,
  `combat_power_snapshot`, `world_unlocked`.
- **Education**: `question_answered` (skill, difficulty, correct, latency),
  `skill_level_up`, `vocabulary_mastered`, `weak_word_logged`,
  `weak_word_reviewed`, `listening_score`, `speaking_assessment`.

Adaptive difficulty (§6) and the Parent Dashboard both read from the
Education stream; game balancing reads from the Gameplay stream.

## 8. Platform notes

- Portrait-only, safe-area-aware UI (notch/gesture-bar handling).
- Touch targets sized for children (minimum 48dp).
- Microphone permission requested contextually, only right before the
  first Speaking challenge, with a clear explanation string — never at
  first app launch.
- All child-safety-relevant network calls (guild chat, leaderboard name
  display) route through a moderation-check step before display, per the
  GDD §14 requirements.
