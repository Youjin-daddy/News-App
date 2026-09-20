# Verbaria: Wordbound Rising — Design Package

This folder contains Phase 1–2 deliverables for an original mobile English-learning
action/idle RPG: the complete Game Design Document, technical architecture, data
model, UI specification, and development roadmap. No proprietary characters, art,
maps, UI, code, or names from any existing game are used or referenced anywhere in
this package — all identifiers below (world names, monster names, item names,
currencies, factions) are original creations for this project.

Per the project brief, production proceeds in phases and does not start with a
full code dump. This package is Phase 1 ("Complete Game Design Document") and
Phase 2 ("Technical architecture"), plus the analysis needed to identify the
smallest playable prototype (Phase 3 scope).

## Contents

| File | Covers |
|---|---|
| [`01-game-design-document.md`](./01-game-design-document.md) | Core concept, gameplay loop, English skill system, combat, items, monsters, worlds, competition, guilds, quests, weak-word system, parent dashboard, monetization, educational principles, originality/legal notes |
| [`02-technical-architecture.md`](./02-technical-architecture.md) | Unity project structure, data-driven architecture, save system, speech-service abstraction, backend-ready design, analytics, adaptive difficulty algorithm |
| [`03-data-model.md`](./03-data-model.md) | Concrete schemas (question bank, items, monsters, worlds, player profile, guild, league) as ScriptableObject/C# class definitions and JSON shapes |
| [`04-ui-specification.md`](./04-ui-specification.md) | Screen-by-screen UI spec for all 9 primary screens plus English-learning sub-screens |
| [`05-roadmap-and-risks.md`](./05-roadmap-and-risks.md) | 12-phase roadmap, MVP scope, smallest playable prototype definition, complexity estimates, risks, recommended implementation order |

## One-line pitch

**Verbaria: Wordbound Rising** — a spreading force called **the Hush** is
draining meaning from the world of Verbaria, twisting broken language into
monsters. You play a **Wordbound**, a hero whose combat power is literally
fueled by language clarity: every vocabulary word learned, grammar rule
mastered, passage understood, sentence heard, and sentence spoken sharpens
your blade and your magic. The better your English, the stronger you are —
not as two separate systems bolted together, but as one system, because in
Verbaria, clear language *is* power.

## Design priority order (per brief, unchanged throughout this package)

1. Learning effectiveness
2. Fun
3. Long-term progression
4. Fair competition
5. Child safety
6. Technical scalability
7. Monetization
