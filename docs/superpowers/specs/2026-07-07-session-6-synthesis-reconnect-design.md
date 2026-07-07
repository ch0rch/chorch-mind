# Session 6 — Weekly Synthesis & Reconnection Radar

**Date:** 2026-07-07
**Status:** Approved (design) — autonomous build authorized by user for S6
**Process:** Superpowers (brainstorm → design → plan → implement)
**Scope:** Session 6 of the chorch-mind build sequence

---

## Context

`chorch-mind` is a personal second brain built session by session. By the end
of Session 5 the vault has four portable skills (`contact-curation`,
`inbox-triage`, `daily-log`, `para-classify`), a queryable CRM, daily
interaction logging that keeps `last_contact` fresh, and a topic-MOC seed. Per
the build sequence in `docs-second-brain/architecture-draft.md`, Session 6 is
**weekly synthesis + reconnection radar**.

The architecture note foresees these running on a schedule (Vercel Cron) once
Eve is deployed. In Claude Code today they are invoked manually; the scheduling
layer is Session 7.

**Governing convention:** `CLAUDE.md` stays minimal; all operational logic
lives in portable, agent-agnostic skills under `.claude/skills/`. Skills must
not hardcode any harness-specific mechanism and preserve
TRIGGER → DO → VERIFY → STOP behavior (the build session was authorized to run
autonomously, but the skills' behavior is unchanged).

## Goal

Give the vault two periodic-review skills: `reconnect-radar`, a read-only skill
that surfaces which people are due for reconnection based on `last_contact` and
`relationship`; and `weekly-synthesis`, which digests the week's daily notes and
CRM movement into a saved review note.

## Non-Goals (YAGNI)

- No automated scheduling. Cron/always-on runs arrive with Eve in Session 7.
  Here both skills are invoked manually.
- No notifications/messaging. Presenting the digest via a channel is Eve's job;
  the skill only produces the content agent-agnostically.
- No new CRM fields. The radar uses existing `last_contact` and `relationship`.
- No auto-mutation from the radar — it is strictly read-only and advisory.

## Key Principles Applied

- **Built on fresh `last_contact`.** S4 keeps it current; S6 consumes it.
- **PARA: review is an ongoing responsibility** — synthesis notes live in a
  dedicated `reviews/` folder, not mixed into `daily/`.
- **`raw` is sacred.** Synthesis reads derived notes; it never touches the inlet.
- **Agent-agnostic + VERIFY → STOP.** `weekly-synthesis` writes a note and so
  confirms before committing; `reconnect-radar` writes nothing (read-only).

## Decisions

1. **Two new portable skills**, not `CLAUDE.md` rules.
2. **`reconnect-radar` is read-only.** It computes days since `last_contact` and
   flags people past a cadence threshold that depends on `relationship`:
   - `close` — reconnect if > 14 days.
   - `warm` — reconnect if > 45 days.
   - `cold` — reconnect if > 90 days.
   Thresholds are orientation, stated in the skill; the human decides.
3. **`weekly-synthesis` writes a review note** to `reviews/YYYY-MM-DD.md`
   summarizing the week: interactions logged (from `daily/`), people contacted,
   notes created/filed, and open threads. It reads the last 7 days of `daily/`.
4. **New `reviews/` folder** (with `.gitkeep`) holds synthesis notes — review is
   an ongoing responsibility, kept separate from daily notes.
5. **`CLAUDE.md` pointer updated** with both skills.

## Design

### `.claude/skills/reconnect-radar/SKILL.md` (new — read-only)

Frontmatter:
```markdown
---
name: reconnect-radar
description: Use to see which people are due for reconnection. Read-only: scans people/ notes, computes days since last_contact, and flags contacts past a cadence threshold based on relationship (close > 14d, warm > 45d, cold > 90d). Suggests who to reach out to; never mutates the vault.
---
```
Body (Spanish) describes the flow:
1. TRIGGER: "¿a quién debería reconectar?" (manual, or scheduled under Eve).
2. Read `people/` notes; for each, compute days since `last_contact` against
   today.
3. Flag the person if the gap exceeds the cadence for their `relationship`
   (close > 14d, warm > 45d, cold > 90d — orientation, not law).
4. Present the list ordered by how overdue each is, with name, days since
   contact, and channel. **Read-only: propose, never write.**

### `.claude/skills/weekly-synthesis/SKILL.md` (new)

Frontmatter:
```markdown
---
name: weekly-synthesis
description: Use to produce a weekly review. Reads the last 7 days of daily/ notes and CRM movement, digests interactions, people contacted, notes created/filed, and open threads, and writes a review note to reviews/<today>.md. Shows the digest and waits for approval before committing (VERIFY then STOP).
---
```
Body (Spanish) describes the flow:
1. TRIGGER: weekly review (manual, or scheduled under Eve).
2. Read the last 7 days of `daily/` notes and recent CRM movement.
3. Build a digest: interactions logged (with `[[Person]]` links), people
   contacted, notes created/filed into PARA, and open threads.
4. Write the digest to `reviews/<today>.md` with the structure below.
5. Never invent activity that is not in the notes. VERIFY → STOP before writing.

Review-note structure:
```markdown
---
type: review
period: <YYYY-MM-DD week>
date: <YYYY-MM-DD>
---

## Interacciones de la semana

## Personas contactadas

## Notas creadas / archivadas

## Hilos abiertos
```

### `reviews/.gitkeep` (new)

Folder for synthesis notes.

### `CLAUDE.md` pointer update

Add `reconnect-radar` and `weekly-synthesis` to the skills list, and note that
review notes live in `reviews/`.

## Data Flow

```
people/*.md (last_contact, relationship) ──reconnect-radar──▶ read-only "who to reconnect" list

daily/ (last 7 days) + CRM movement ──weekly-synthesis──▶ reviews/<today>.md (digest)
```

## Testing / Verification

- Both SKILL.md files have valid frontmatter and load.
- `reconnect-radar` states the three cadence thresholds and is marked read-only.
- `weekly-synthesis` states the review-note structure and preserves VERIFY→STOP.
- `reviews/` exists and is tracked (`.gitkeep`).
- `CLAUDE.md` lists both new skills and mentions `reviews/`.

## Acceptance Criteria

- `reconnect-radar` (read-only) and `weekly-synthesis` skills exist, portable
  and agent-agnostic.
- The radar uses existing `last_contact`/`relationship`; no new fields.
- Synthesis writes to `reviews/`, kept separate from `daily/`.
- `CLAUDE.md` pointer updated, still minimal.
- Skill behavior preserves TRIGGER → DO → VERIFY → STOP (synthesis writes;
  radar is read-only).
