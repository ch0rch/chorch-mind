# Session 4 — Daily Notes & Interaction Backlinks

**Date:** 2026-07-07
**Status:** Approved (design)
**Process:** Superpowers (brainstorm → design → plan → implement)
**Scope:** Session 4 of the chorch-mind build sequence

---

## Context

`chorch-mind` is a personal second brain built session by session. By the end
of Session 3 the vault has portable skills for contact curation and inbox
triage, and a queryable `people/` CRM. Per the build sequence in
`docs-second-brain/architecture-draft.md`, Session 4 is **backlinks + `daily/`
linking interactions to people**.

Today interactions are recorded inside each person note under a `## Historial`
section, and `daily/` is empty. Session 4 moves interactions to daily notes and
relies on Obsidian backlinks, so a person note surfaces its interactions
automatically instead of duplicating them.

**Governing convention:** `CLAUDE.md` stays minimal; all operational logic
lives in portable, agent-agnostic skills under `.claude/skills/`
(see the skills-portability convention). Skills must not hardcode any
harness-specific ask mechanism.

## Goal

Make `daily/` the single source of truth for interactions. An interaction is
logged in `daily/YYYY-MM-DD.md` with a `[[Person]]` backlink; the person note
surfaces it via Obsidian's linked mentions (no duplication); the person's
`last_contact` is updated automatically. Retire the manual `## Historial`.

## Non-Goals (YAGNI)

- No reconnection radar / "who to reconnect with" — that is Session 6 (this
  session only keeps `last_contact` fresh so S6 can build on it).
- No auto-generated Historial section inside person notes — backlinks replace it.
- No tight coupling to `inbox-triage`. `daily-log` is invoked directly on a
  natural-language interaction. Triage may route day-notes to `daily/` later,
  but that integration is out of scope here.
- No PARA classification of day notes.

## Key Principles Applied

- **DRY via backlinks.** `daily/` holds the interaction once; the person note
  shows it through linked mentions. No copy into the person note.
- **`raw` is sacred**; person notes and daily notes are derived/editable.
- **TRIGGER → DO → VERIFY → STOP** before every write.
- **Agent-agnostic**: the skill describes what to record and confirm, never the
  harness mechanism.
- **Dedupe reuse**: person matching reuses the name/aliases logic already used
  by `contact-curation`.

## Decisions

1. **Source of truth = `daily/`** (user decision). The `## Historial` section is
   retired from the template and from existing notes.
2. **`daily-log` updates `last_contact` automatically** (user decision) when it
   logs an interaction — this is what keeps the CRM current and feeds S6.
3. **Migrate existing data** (user decision): move `fede-olgue.md`'s Historial
   entry into `daily/2026-07-07.md`, then remove the `## Historial` section from
   that note. `last_contact` stays `2026-07-07`.
4. **New portable skill `daily-log`**, not `CLAUDE.md` rules.

## Design

### `daily/YYYY-MM-DD.md` structure

```markdown
---
type: daily
date: 2026-07-07
---

## Interacciones

- [[Fede Olgue]] — Primera charla por Meet. Quiere sesiones 1-1 sobre IA.

## Notas del día

<anything that is not an interaction with a person>
```

- `## Interacciones` — one bullet per interaction, each starting with a
  `[[Person]]` backlink, then a short summary.
- `## Notas del día` — free-form day notes not tied to a person.

### `.claude/skills/daily-log/SKILL.md` (new)

Frontmatter:
```markdown
---
name: daily-log
description: Use when logging an interaction with a person (a meeting, call, message). Appends the interaction to daily/<today>.md with a [[Person]] backlink, matching the person in people/ by name/aliases, and updates that person's last_contact. Creates the day note if missing. Shows the change and waits for approval before committing (VERIFY then STOP).
---
```
Body (Spanish, matching the vault) describes the flow:
1. TRIGGER: an interaction in natural language.
2. Match the person in `people/` (dedupe by `name`/`aliases`; if no match, say
   so — do not invent a person).
3. Append a bullet to `daily/<today>.md` under `## Interacciones` with
   `[[Person]]` + summary. Create the day note from the structure above if it
   does not exist.
4. Update that person's `last_contact` to today's date.
5. Never invent fields. VERIFY → STOP before writing.

### Migration (touches Session 1 artifacts)

- `people/_template.md`: remove the `## Historial` section (keep `## Notas`).
- `people/fede-olgue.md`: remove its `## Historial` section; its one entry moves
  to `daily/2026-07-07.md` as an `[[Fede Olgue]]` interaction. `last_contact`
  unchanged (`2026-07-07`).
- `daily/2026-07-07.md`: created with Fede's migrated interaction.

## Data Flow

```
interaction (NL)  ──daily-log──▶  daily/<today>.md  (## Interacciones: [[Person]] — summary)
                                        │
                                        ├─▶ Obsidian linked mentions surface it in people/<slug>.md
                                        └─▶ people/<slug>.md last_contact := today
```

## Testing / Verification

- `daily-log` SKILL.md has valid frontmatter and loads.
- `daily/2026-07-07.md` exists with `type: daily`, a `## Interacciones` section,
  and the `[[Fede Olgue]]` migrated entry.
- `people/fede-olgue.md` no longer has a `## Historial` section; no interaction
  data lost (it lives in `daily/2026-07-07.md`).
- `people/_template.md` no longer has `## Historial`.
- `people/fede-olgue.md` `last_contact` is still `2026-07-07`.

## Acceptance Criteria

- `daily-log` skill exists, portable and agent-agnostic.
- A day note holds interactions with `[[Person]]` backlinks; person notes are
  not duplicated.
- Logging an interaction updates the person's `last_contact`.
- `## Historial` is retired from the template and from `fede-olgue.md`, with its
  existing data migrated to `daily/`.
- Everything follows TRIGGER → DO → VERIFY → STOP.
