# Data Model — Verbaria: Wordbound Rising

Concrete schemas for the content types referenced in the GDD and
architecture docs. These are class/schema *definitions* for the data-driven
architecture (Phase 2), not gameplay logic — no behavior is implemented
here.

## 1. Question (English content bank)

Every question, across all five skills, shares one metadata envelope so
the same adaptive-difficulty, weak-word, and league-scoring code can treat
them uniformly. Stored as versioned JSON (`StreamingAssets/Questions/`),
not hard-coded, so content can be extended (including AI-assisted authoring,
§26) without an app rebuild.

```json
{
  "question_id": "voc_00042",
  "skill": "vocabulary",
  "grade_level": 6,
  "cefr_level": "B1",
  "difficulty": 3,
  "question_type": "multiple_choice_meaning",
  "question": "What does 'reluctant' mean?",
  "choices": ["unwilling", "excited", "angry", "careless"],
  "answer": "unwilling",
  "explanation": "'Reluctant' means unwilling or hesitant to do something.",
  "target_vocabulary": ["reluctant"],
  "grammar_point": null,
  "audio": null,
  "speaking_target": null,
  "review_interval_days": [1, 3, 7, 16, 35],
  "content_status": "approved",
  "content_source": "human_authored"
}
```

Field notes:

| Field | Type | Notes |
|---|---|---|
| `question_id` | string | Stable, never reused |
| `skill` | enum | `vocabulary`\|`grammar`\|`reading`\|`listening`\|`speaking` |
| `grade_level` | int | Approximate school-grade equivalent, for parent-facing display |
| `cefr_level` | enum | `A1`–`C1`, drives adaptive-difficulty banding |
| `difficulty` | int 1–10 | Fine-grained band within a CEFR level |
| `question_type` | enum | e.g. `multiple_choice_meaning`, `synonym`, `antonym`, `fill_blank`, `spelling`, `word_matching`, `context_usage`, `sentence_correction`, `sentence_completion`, `main_idea`, `inference`, `detail`, `listening_choice`, `speaking_prompt` |
| `choices` | string[] \| null | Null for spelling/speaking/free-response types |
| `answer` | string | Canonical answer key |
| `explanation` | string | Always shown on incorrect answer |
| `target_vocabulary` | string[] | Feeds Weak-Word Ledger and vocabulary mastery tracking |
| `grammar_point` | string \| null | e.g. `passive_voice`, `conditional_type_2` |
| `audio` | string \| null | Clip reference (listening questions); null triggers on-device TTS fallback |
| `speaking_target` | object \| null | See §2 below |
| `review_interval_days` | int[] | Spaced-repetition schedule if missed |
| `content_status` | enum | `draft`\|`pending_review`\|`approved`\|`retired` — gates visibility to players (§26) |
| `content_source` | enum | `human_authored`\|`ai_generated` — provenance tracking, required for moderation audit |

## 2. SpeakingTarget (embedded in a speaking-type Question)

```json
{
  "prompt_audio": "npc_where_going.mp3",
  "prompt_text": "Where are you going?",
  "target_sentence": "I am going to school.",
  "accepted_patterns": [
    "I am going to school.",
    "I'm going to school.",
    "I'm going to school now."
  ],
  "required_slots": { "verb": "go", "tense": "present_continuous", "destination_present": true },
  "scoring_weights": { "pronunciation": 0.25, "accuracy": 0.35, "fluency": 0.20, "context": 0.20 }
}
```

`accepted_patterns` is a seed list for fuzzy/slot matching, not an
exhaustive exact-match list — evaluated by `ISpeechRecognitionService`
(architecture doc §5).

## 3. WeakWordLedgerEntry (per-player, per-item)

```json
{
  "item_id": "reluctant",
  "skill": "vocabulary",
  "times_missed": 2,
  "times_reviewed_correct": 1,
  "next_review_due": "2026-09-25T00:00:00Z",
  "mastery_state": "learning"
}
```

`mastery_state`: `new → learning → reviewing → mastered`, driving the
Profile screen's "Words Mastered" vs "Weak Words" counts (GDD §23).

## 4. ItemDefinition

```json
{
  "item_id": "wyrmfang_blade",
  "display_name": "Wyrmfang Blade",
  "slot": "weapon",
  "rarity": "epic",
  "level_requirement": 30,
  "base_stats": { "atk": 180 },
  "upgrade_curve": { "max_level": 10, "gold_per_level": 500, "runeshards_per_level": 3 },
  "socket_count": 2,
  "english_effect": {
    "trigger": "vocabulary_streak",
    "threshold": 5,
    "effect": "next_attack_damage_multiplier",
    "value": 3.0,
    "effect_name": "Wyrm Strike"
  }
}
```

## 5. MonsterDefinition

```json
{
  "monster_id": "bonemouth_knight",
  "display_name": "Bonemouth Knight",
  "world_id": "bastion_of_broken_verbs",
  "hp": 4200,
  "attack": 220,
  "defense": 140,
  "speed": 90,
  "element": "umbral",
  "weakness": "runic",
  "loot_table": {
    "gold_range": [80, 140],
    "drops": [
      { "item_id": "runeshard_common", "weight": 60 },
      { "item_id": "bastionguard_sword", "weight": 8 },
      { "item_id": "wyrmfang_blade", "weight": 1 }
    ]
  }
}
```

## 6. WorldDefinition

```json
{
  "world_id": "wyrmreach",
  "display_name": "Wyrmreach",
  "order": 7,
  "english_focus": ["vocabulary", "grammar", "reading", "listening", "speaking"],
  "unlock_requirements": {
    "min_equipment_power": 1000,
    "min_skill_levels": { "reading": 20, "listening": 20, "speaking": 15 },
    "required_boss_cleared": "bastion_of_broken_verbs_boss"
  },
  "monster_ids": ["cindertongue_wyrm", "elder_wyrm", "wyrm_sovereign"]
}
```

## 7. PlayerEnglishProfile (per-player runtime state)

```json
{
  "player_id": "device_local_0001",
  "skills": {
    "vocabulary": { "level": 25, "xp": 12500, "rolling_accuracy": 0.91 },
    "grammar":    { "level": 20, "xp": 9800,  "rolling_accuracy": 0.68 },
    "reading":    { "level": 23, "xp": 11000, "rolling_accuracy": 0.80 },
    "listening":  { "level": 18, "xp": 7200,  "rolling_accuracy": 0.75 },
    "speaking":   { "level": 16, "xp": 6100,  "rolling_accuracy": 0.72 }
  },
  "weak_word_ledger": ["reluctant", "although", "..."],
  "words_mastered_count": 340
}
```

## 8. PlayerSaveData (top-level save document)

```json
{
  "player_id": "device_local_0001",
  "character": { "level": 34, "xp": 154200, "combat_power": 2380 },
  "english_profile": { "$ref": "PlayerEnglishProfile" },
  "inventory": [ { "item_id": "wyrmfang_blade", "instance_id": "inst_991", "level": 6, "sockets": ["runeshard_common"] } ],
  "equipped": { "weapon": "inst_991", "armor": null, "helmet": null, "boots": null, "accessory_1": null, "accessory_2": null },
  "gold": 4820,
  "runeshards": 63,
  "quests_active": ["defeat_20_goblins"],
  "quests_completed": ["tutorial_01"],
  "worlds_unlocked": ["whisperwood", "duskmire_thicket"],
  "guild_id": null,
  "settings": { "social_features_enabled": true, "parent_pin_hash": null },
  "save_version": 1
}
```

`save_version` supports forward migration when the schema changes.

## 9. LeagueEntry (per-player, per-season)

```json
{
  "player_id": "device_local_0001",
  "season_id": "2026-W38",
  "tier": "intermediate",
  "league_points": 4820,
  "score_breakdown": {
    "accuracy_weighted": 1800,
    "difficulty_bonus": 900,
    "new_words_learned": 220,
    "consecutive_streak_bonus": 400,
    "combat_progress_delta": 1500
  }
}
```

## 10. GuildDefinition

```json
{
  "guild_id": "knights_of_clarity",
  "display_name": "Knights of Clarity",
  "member_ids": ["device_local_0001", "..."],
  "total_guild_xp": 6700,
  "castle_tier": 2,
  "social_settings": { "chat_mode": "preset_phrases_only" }
}
```

## 11. ScriptableObject vs. JSON boundary

| Content type | Storage | Rationale |
|---|---|---|
| ItemDefinition, MonsterDefinition, WorldDefinition, SkillDefinition, RewardTable, QuestDefinition | Unity ScriptableObject | Tightly coupled to prefabs/visuals, edited in-Editor, ships with the build |
| Question bank, SpeakingTarget | Versioned JSON in StreamingAssets, later a remote content service | Needs frequent updates without app-store releases; needs AI-assisted authoring + moderation pipeline (§26 of GDD) independent of app versioning |
| PlayerSaveData, PlayerEnglishProfile, WeakWordLedgerEntry, LeagueEntry, GuildDefinition | Runtime JSON via `ISaveRepository` / future backend | Per-player mutable state |
