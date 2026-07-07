# Session 1 — Vault Foundation & People Contract

**Date:** 2026-07-07
**Status:** Approved (design)
**Process:** Superpowers (brainstorm → design → plan → implement)
**Scope:** Session 1 of the chorch-mind build sequence

---

## Context

`chorch-mind` is a personal second brain built session by session. It doubles
as the replicable pilot for a consulting client. The focus is
networking / contact management (a CRM) on top of an Obsidian vault.

**Target architecture (macro, unchanged):**

- **GitHub private repo** = source of truth (the vault under git; git history is
  the undo / safety net against an autonomous agent).
- **Obsidian local** (Obsidian Git plugin) = the UI where the human reads/edits.
- **Eve @ Vercel** = durable always-on agent, Telegram/Slack channel. It is the
  primary contact inlet (natural-language contacts curated into the CRM).
- **Claude Code** = deep-work bench for building/refactoring the vault.
- **GitHub** = the sync hub for all actors.

Session 1 lays the foundation only. Eve arrives later (Session 7); the
NL → curate → store pattern is first practiced with Claude Code from the
desktop, then migrated to Eve.

## Goal

Establish the vault skeleton and the root `CLAUDE.md` (the rules the agent
obeys), plus the person-note contract, so that by the end of Session 1 the CRM
is alive: the first real contact can be loaded by talking to Claude Code in
natural language.

## Non-Goals (YAGNI)

- No database (Postgres/Supabase). The CRM is markdown-in-repo. The frontmatter
  schema is DB-ready so a future migration (frontmatter → columns) stays
  mechanical, but no DB is built now.
- No encryption layer (git-crypt). The private repo is sufficient for a personal
  second brain. git-crypt can be added later without pain.
- No importer (LinkedIn/CSV/Excel). The inlet is conversational, not batch.
- No Eve deployment, no MCP wiring, no classification skills. Those are later
  sessions.

## Deliverables

### 1. Vault skeleton

```
chorch-mind/
├── CLAUDE.md          # the rules the agent obeys
├── inbox/
│   └── raw/           # the single inlet; raw is sacred, never edited
├── people/            # the CRM
│   └── _template.md   # the person-note contract
├── daily/             # daily notes
├── projects/          # PARA
├── areas/
├── resources/
└── archive/
```

Empty directories get a `.gitkeep` so git tracks them.

### 2. Person-note contract (the core)

One note per person at `people/<slug>.md`. `<slug>` is a kebab-case of the
person's name (e.g. `juan-perez.md`).

Frontmatter schema (each field maps to a future Postgres column):

```yaml
---
type: person
name: Juan Pérez
aliases: [Juanchi]
company: Fintech XYZ
role: Head of Product
location: Buenos Aires
tags: [fintech, product]
met_via: "[[Mengano García]]"      # backlink to who introduced them
met_context: "Conferencia de producto"
first_met: 2026-10-15
last_contact: 2026-10-15
relationship: warm                  # cold | warm | close
channels: { linkedin: "", email: "", phone: "" }
---

## Notas

## Historial
- 2026-10-15 — Nos presentó [[Mengano García]] en la conf de producto.
```

Rationale:

- Every frontmatter field is a future DB column → migration stays mechanical.
- `met_via` plus `[[...]]` links in the body are the live graph — the backlinks
  that make this a second brain and not a spreadsheet.
- Note **body content is the user's** and stays in Spanish; only the schema keys
  are English identifiers.

### 3. Curation rules (live in root `CLAUDE.md`)

What any agent follows when it receives a contact in natural language:

1. Raw NL enters `inbox/raw/` and is **sacred** — never edited.
2. The agent **extracts** structured fields and creates/updates
   `people/<slug>.md`.
3. **Dedupe** by `name` / `aliases` before creating a new note.
4. If the user mentions who introduced the contact → set `met_via` as a
   `[[backlink]]`.
5. **Never invent fields.** Unknown values are omitted, not filled with
   placeholders.
6. **VERIFY:** the agent shows the proposed note and **waits for explicit OK**
   before committing. STOP. No auto-merge.

### 4. Root `CLAUDE.md` scope

The root `CLAUDE.md` documents:

- What PARA is and how `projects/areas/resources/archive` are used
  (organize by actionability, not by topic).
- `inbox/raw` is the single inlet and is sacred.
- The operating loop: **TRIGGER → DO → VERIFY → STOP** (never auto-commit
  without the human's OK).
- "Keys not prompts" — permissions over trust.
- The curation rules above.

## Validation (definition of done for S1)

Load the **first real contact** by talking to Claude Code in natural language.
Observe the agent curate it per the rules, approve the proposed note, and commit
it. Session 1 closes with the CRM containing one real person and the foundation
in place.

## Operating Constraints

- Human-in-the-loop always. No autonomous commits in this session.
- Pull-before-write + atomic commits (three actors will eventually push to the
  same repo).
- Build is pedagogical and incremental: understand each piece before the next.

## Open Questions

None blocking. Future decision points (not part of S1): Supabase migration
triggers (relational queries Dataview can't serve, thousands of contacts,
external integrations, multi-user), and whether git-crypt is warranted later.
