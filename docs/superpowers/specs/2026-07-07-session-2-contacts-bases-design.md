# Session 2 — Contacts Module & Bases

**Date:** 2026-07-07
**Status:** Approved (design)
**Process:** Superpowers (brainstorm → design → plan → implement)
**Scope:** Session 2 of the chorch-mind build sequence

---

## Context

`chorch-mind` is a personal second brain built session by session. Session 1
laid the vault foundation: the PARA + `inbox/raw` skeleton, the root
`CLAUDE.md` rules, the person-note contract (`people/_template.md`), and the
first curated contact (`people/fede-olgue.md`).

Per the build sequence in `docs-second-brain/architecture-draft.md`, Session 2
is the **contacts module (`people/`) + Bases**: make the CRM not just a pile of
notes but a *queryable* directory.

## Goal

Turn `people/` into a usable, queryable contacts module by adding an Obsidian
Base that renders the CRM as a directory grouped by **company** and by **role**
— the "targeted networking" use case (who works where).

## Non-Goals (YAGNI)

- No reconnection radar / "who to reconnect with" views — that is Session 6.
- No relationship-level filtering or connector-network views — not requested.
- No classification skills or MCP — that is Session 3.
- No Dataview — the view engine is native Obsidian Bases (declarative,
  plugin-free, versions cleanly in git).
- No formulas beyond the empty-value fallbacks below.

## Decisions

1. **View engine: native Obsidian Bases** (1.9+). Declarative `.base` YAML,
   no third-party plugin, plain text under git.
2. **One `.base` file, two views.** A single `people/contacts.base` holds a
   "Por empresa" view and a "Por rol" view. Bases supports multiple views per
   file.
3. **Empty values group explicitly.** Contacts with no `company` group under
   **"Sin empresa"**; no `role` groups under **"Sin rol"**. Achieved with
   fallback formulas, not raw fields — honors the "never invent fields" rule
   (the note stays empty; only the view labels it).
4. **Template excluded via `note.name` filter.** `_template.md` has an empty
   `name:`, so a `note.name` truthiness filter keeps it out of the views
   without needing a separate folder or template edit.

## Design

### `people/contacts.base` (new)

```yaml
formulas:
  company_group: 'if(note.company, note.company, "Sin empresa")'
  role_group: 'if(note.role, note.role, "Sin rol")'

filters:
  and:
    - file.inFolder("people")
    - 'note.type == "person"'
    - 'note.name'          # excludes _template.md (empty name)

views:
  - type: table
    name: "Por empresa"
    groupBy:
      property: formula.company_group
    order: [note.name, note.role, note.relationship, note.last_contact]

  - type: table
    name: "Por rol"
    groupBy:
      property: formula.role_group
    order: [note.name, note.company, note.relationship, note.last_contact]
```

### `people/_template.md` (review only)

The two grouping fields — `company` and `role` — already exist in the template
frontmatter. Expected outcome: **no change**; confirm consistency only.

### `people/README.md` (new)

Short usage doc:
- How a contact enters the CRM: raw text lands in `inbox/raw/` (sacred,
  untouched); the agent curates it into `people/<slug>.md` per the template.
- How to read the two views of the Base in Obsidian.

## Data Flow

```
inbox/raw/<contact>.md   →  agent curates  →  people/<slug>.md (frontmatter)
                                                     │
                                                     ▼
                                          people/contacts.base
                                          ├── "Por empresa"  (groupBy company_group)
                                          └── "Por rol"      (groupBy role_group)
```

## Testing / Verification

- Open the vault in Obsidian; `contacts.base` renders both views.
- `_template.md` does NOT appear in either view (name filter works).
- `fede-olgue.md` appears under its company (or "Sin empresa" if none).
- A synthetic contact with no `company` lands under "Sin empresa".

## Acceptance Criteria

- `people/contacts.base` exists with both views and the fallback formulas.
- The template is confirmed consistent (or minimally adjusted).
- `people/README.md` documents the add-contact flow and how to read the Base.
- The change follows TRIGGER → DO → VERIFY → STOP: shown and OK'd before commit.
