# Session 5 — PARA Classification & Topic Wiki

**Date:** 2026-07-07
**Status:** Approved (design) — autonomous build authorized by user for S5 only
**Process:** Superpowers (brainstorm → design → plan → implement)
**Scope:** Session 5 of the chorch-mind build sequence

---

## Context

`chorch-mind` is a personal second brain built session by session. By the end
of Session 4 the vault has three portable skills (`contact-curation`,
`inbox-triage`, `daily-log`), a queryable `people/` CRM, and daily interaction
logging. Per the build sequence in `docs-second-brain/architecture-draft.md`,
Session 5 is **PARA + topic wiki**.

Session 3 deliberately left PARA classification as *non-binding hints*: the
triage tags an item `project? / area? / resource?` and asks the user where it
goes. Session 5 **systematizes** that decision into a portable skill with
explicit criteria, and introduces the first "wiki by topic" primitive.

**Governing convention:** `CLAUDE.md` stays minimal; all operational logic
lives in portable, agent-agnostic skills under `.claude/skills/`. Skills must
not hardcode any harness-specific ask mechanism, and they preserve
TRIGGER → DO → VERIFY → STOP behavior even though this build session was
authorized to run autonomously.

## Goal

Give the vault a systematic PARA classifier (`para-classify`) that routes an
item to `projects/`, `areas/`, `resources/`, or `archive/` by actionability,
and seed the topic wiki with a Map-of-Content (MOC) note template so related
notes can be grouped by topic via backlinks.

## Non-Goals (YAGNI)

- No automatic, unattended re-filing of the whole vault. The classifier
  proposes; the human confirms (VERIFY → STOP), as always.
- No full topic-wiki engine or auto-linking. Session 5 seeds the pattern with a
  MOC template; a heavier wiki skill is deferred until the user weighs in.
- No change to `inbox-triage`'s hint behavior. `para-classify` consumes the
  hint; it does not replace triage.
- No new PARA folders. `projects/ areas/ resources/ archive/` already exist.

## Key Principles Applied

- **PARA sorts by actionability, not topic.** The criteria below come straight
  from `CLAUDE.md`.
- **Notes flow.** Classification is never permanent; `para-classify` can
  re-evaluate an already-filed note and propose moving it as its actionability
  changes.
- **`raw` is sacred.** Items being classified are derived notes (e.g. from
  `inbox/triaged/`); the raw inlet is never touched.
- **DRY with S3.** `para-classify` reuses the `para_hint` produced by
  `inbox-triage` as orientation, not as a verdict.
- **Agent-agnostic + VERIFY → STOP** preserved in the skill body.

## Decisions

1. **New portable skill `para-classify`**, not `CLAUDE.md` rules.
2. **Classification criteria (verbatim from `CLAUDE.md` structure):**
   - `projects/` — has an objective **and** an end date.
   - `areas/` — an ongoing responsibility with no end date.
   - `resources/` — a topic of interest that is not a responsibility.
   - `archive/` — dead (closed project, dropped area).
3. **Input** is typically a note in `inbox/triaged/` (carrying a `para_hint`
   and `form` from S3) or any existing note to re-classify. Output is the note
   moved to the chosen PARA folder; the `raw` stays intact.
4. **Topic wiki = MOC template** at `resources/_topic-template.md`. A topic note
   is an index that links related notes by `[[backlink]]`. This is the minimal
   seed of "wiki por tema"; no classifier or auto-linker is built for it yet.
5. **`CLAUDE.md` pointer updated** to list `para-classify` and mention the topic
   MOC, keeping the root file minimal.

## Design

### `.claude/skills/para-classify/SKILL.md` (new)

Frontmatter:
```markdown
---
name: para-classify
description: Use when filing a note into PARA (projects/areas/resources/archive) or re-classifying one whose actionability changed. Classifies by actionability using the PARA criteria, consuming any para_hint from inbox-triage as orientation, proposes a destination with a reason, and moves the note on approval. raw stays sacred; VERIFY then STOP.
---
```
Body (Spanish) describes the flow:
1. TRIGGER: a note to file into PARA (from `inbox/triaged/` or an existing note
   to re-evaluate).
2. Apply the PARA criteria by actionability (objective+end-date → projects;
   ongoing responsibility → areas; interest without action → resources; dead →
   archive). Use any `para_hint` as orientation, not a verdict.
3. Propose the destination **with a one-line reason**. Because notes flow, this
   may mean moving a note already filed elsewhere.
4. On approval, move the note to the chosen PARA folder (keep its backlinks;
   `raw` untouched). Never invent fields.
5. VERIFY → STOP before moving anything.

### `resources/_topic-template.md` (new — topic wiki seed)

```markdown
---
type: topic
title:
tags: []
---

## Qué es

## Notas relacionadas

## Referencias
```
A topic note (MOC) is an index that groups related notes by topic using
`[[backlinks]]` under "Notas relacionadas".

### `CLAUDE.md` pointer update

Extend the "Lógica operativa: skills portables" list with `para-classify`, and
add a one-line mention that topic notes (MOC) live in `resources/` from
`resources/_topic-template.md`.

## Data Flow

```
inbox/triaged/<note>  (form + para_hint from S3)
        │
        ▼
   para-classify  ──(by actionability)──▶  projects/ | areas/ | resources/ | archive/
                                                  │
   resources/_topic-template.md ──▶ topic MOC note ──[[backlinks]]──▶ related notes
```

## Testing / Verification

- `para-classify` SKILL.md has valid frontmatter and loads.
- The skill body states all four PARA criteria and the "notes flow /
  re-classify" behavior, and preserves VERIFY → STOP.
- `resources/_topic-template.md` exists with `type: topic` frontmatter and the
  three sections.
- `CLAUDE.md` lists `para-classify` and mentions the topic MOC.

## Acceptance Criteria

- `para-classify` skill exists, portable and agent-agnostic, with the four PARA
  criteria and re-classification behavior.
- Topic MOC template exists as the wiki seed.
- `CLAUDE.md` pointer updated, still minimal.
- Skill behavior preserves TRIGGER → DO → VERIFY → STOP.
