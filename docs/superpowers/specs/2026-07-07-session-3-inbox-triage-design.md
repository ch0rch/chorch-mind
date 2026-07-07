# Session 3 — Inbox Triage & Portable Curation Skills

**Date:** 2026-07-07
**Status:** Approved (design)
**Process:** Superpowers (brainstorm → design → plan → implement)
**Scope:** Session 3 of the chorch-mind build sequence

---

## Context

`chorch-mind` is a personal second brain built session by session. By the end
of Session 2 the CRM is queryable (`people/` + `contacts.base`). Per the build
sequence in `docs-second-brain/architecture-draft.md`, Session 3 is the
**first classification skill**: triage of the inbox.

Today the root `CLAUDE.md` assumes everything landing in `inbox/raw/` is a
contact — its curation rules only cover people. Session 3 generalizes the
inlet: what arrives can be a contact, a note/idea, a task, or a reference.

**Governing convention (this project):** `CLAUDE.md` stays minimal; all
operational logic lives in **portable, agent-agnostic skills** under
`.claude/skills/`, so the same skills run in Claude Code today and in Eve
(the durable agent, Session 7) later.

## Goal

Triage each raw inbox item into its coarse form, route contacts to the
existing curation flow automatically, and for everything else present the user
with a PARA hint plus destination options — the user decides where it goes.
Extract the existing contact-curation rules out of `CLAUDE.md` into a portable
skill, and shrink `CLAUDE.md`.

## Non-Goals (YAGNI)

- No automated PARA classification. The triage proposes a *hint* and asks; the
  human decides. Systematized PARA + topic wiki is Session 5.
- No MCP source ingestion (Gmail/Calendar). The inlet is `inbox/raw/` files.
  Connecting external sources is a later, separate session.
- No redesign of the curation behavior. Moving it to a skill must preserve
  identical behavior — same six rules, verbatim.
- No task-management system. A "task" form is just a hint on a staged note.

## Key Principles Applied

- **PARA flows, nothing is fixed.** A note is not permanently a "note" — it may
  become a project or an area document over time. The triage therefore never
  assigns a final PARA category; it records a non-binding hint and the user's
  chosen destination. ("Manual first, automate later" —
  `architecture-draft.md`.)
- **`raw` is sacred.** The original in `inbox/raw/` is never edited or deleted.
  Triage produces a *derived* note with a backlink.
- **TRIGGER → DO → VERIFY → STOP.** The triage proposes; it writes nothing
  until the user confirms the destination.
- **Agent-agnostic prompting.** The skill describes *what* to ask the user and
  *what* options to present, never *how*. Claude Code asks via the prompt; Eve
  asks via its channel (Telegram/Slack). No harness-specific mechanism is
  hardcoded.

## Decisions

1. **Two portable skills**, not `CLAUDE.md` rules:
   - `.claude/skills/contact-curation/SKILL.md` — the six curation rules, moved
     verbatim from `CLAUDE.md`.
   - `.claude/skills/inbox-triage/SKILL.md` — the classifier.
2. **`CLAUDE.md` shrinks**: the "Reglas de curación de contactos" section is
   removed and replaced by a short pointer to `.claude/skills/`.
3. **Contact has a clear destination** (`people/`) and is routed automatically
   by delegating to `contact-curation`. All other forms require the user to
   choose the destination.
4. **Staging is one option, not the default.** `inbox/triaged/` is the "not sure
   yet" holding option among `projects/ · areas/ · resources/ · inbox/triaged/ ·
   discard`.

## Design

### `.claude/skills/contact-curation/SKILL.md` (new — move, no redesign)

Frontmatter:
```markdown
---
name: contact-curation
description: Use when a contact arrives in natural language or lands in inbox/raw/. Extracts fields into people/<slug>.md from people/_template.md, dedupes by name/aliases, records met_via backlinks, never invents fields, and shows the proposed note for approval before committing (VERIFY→STOP).
---
```
Body = the six existing curation rules, moved verbatim from `CLAUDE.md`:
1. raw text stays intact in `inbox/raw/`;
2. extract fields, create/update `people/<slug>.md` from the template;
3. dedupe by `name`/`aliases` before creating;
4. record who introduced the contact in `met_via` as `[[Name]]` backlink;
5. never invent fields — omit the unknown;
6. VERIFY → STOP: show the proposed note and wait for OK before committing.

### `.claude/skills/inbox-triage/SKILL.md` (new — the classifier)

Frontmatter:
```markdown
---
name: inbox-triage
description: Use when raw items land in inbox/raw/ and need routing. Detects each item's coarse form (contact, note/idea, task, reference); routes contacts to the contact-curation skill automatically; for everything else presents a PARA hint and destination options and lets the user choose. raw stays sacred; VERIFY→STOP before writing.
---
```
Body describes the flow:
1. Read an item from `inbox/raw/`.
2. Detect its **form**: contact / note-idea / task / reference.
3. If **contact** → invoke the `contact-curation` skill (destination `people/`,
   no question needed).
4. If **not a contact**:
   - compute a non-binding **PARA hint** (`project?` / `area?` / `resource?`);
   - present to the user: what was detected + the hint + destination options
     `projects/ · areas/ · resources/ · inbox/triaged/ · discard`;
   - wait for the user's choice (agent-agnostic — do not hardcode the ask);
   - create the derived note in the chosen destination with the frontmatter
     below; leave `raw` intact; add a `source` backlink.
5. VERIFY → STOP is inherent: nothing is written until the user chooses.

Derived-note frontmatter:
```markdown
---
form: note | task | reference
para_hint: project | area | resource
destination: <user's choice>
source: "[[<raw-filename-without-ext>]]"
triaged: <YYYY-MM-DD>
---
<summary/extract of the item>
```

### `CLAUDE.md` (shrink)

Remove the "Reglas de curación de contactos" section. Replace with a short
pointer, e.g. *"La lógica operativa (curación de contactos, triage del inbox)
vive en skills portables — ver `.claude/skills/`."* Keep structure, the
TRIGGER→DO→VERIFY→STOP loop, and the "keys not prompts" principle.

### `inbox/triaged/.gitkeep` (new)

Staging folder for the "not sure yet" destination option.

## Data Flow

```
inbox/raw/<item>.md  (sacred, untouched)
        │
        ▼
   inbox-triage  ── contact ──▶ contact-curation ─▶ people/<slug>.md
        │
        └─ note/task/reference ─▶ [hint + ask user for destination]
                                        │
                                        ▼
                       projects/ | areas/ | resources/ | inbox/triaged/ | discard
                       (derived note, backlink to raw)
```

## Testing / Verification

- Both `SKILL.md` files have valid frontmatter (`name`, `description`) and load.
- `contact-curation` body contains all six original rules (diff against the
  removed `CLAUDE.md` section — no rule lost, no rule added).
- `CLAUDE.md` no longer contains the curation section and points to
  `.claude/skills/`.
- `inbox/triaged/` exists and is tracked (`.gitkeep`).
- Dry run: given a sample non-contact raw item, the triage presents a hint and
  options and writes nothing until a destination is chosen.

## Acceptance Criteria

- `contact-curation` and `inbox-triage` skills exist, portable and
  agent-agnostic (no hardcoded ask mechanism).
- Contact curation behavior is unchanged (rules preserved verbatim).
- `CLAUDE.md` is shrunk and points to the skills.
- Non-contact items yield a PARA hint + destination options; the user chooses;
  `raw` stays intact; the derived note backlinks to the raw.
- Everything follows TRIGGER → DO → VERIFY → STOP.
