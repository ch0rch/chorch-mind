# Session 2 — Contacts Module & Bases Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the `people/` CRM queryable by adding an Obsidian Base that groups contacts by company and by role.

**Architecture:** One declarative `.base` file (native Obsidian Bases, plugin-free) with two table views and fallback formulas for empty values. A usage README documents the add-contact flow and how to read the views. The person-note template is confirmed consistent, not rewritten.

**Tech Stack:** Obsidian Bases (1.9+), YAML frontmatter, Markdown, git.

## Global Constraints

- Source of truth is the git repo; every change is committed. (`CLAUDE.md`)
- Operating loop is TRIGGER → DO → VERIFY → STOP. Show the change and wait for OK before committing. Never auto-merge. (`CLAUDE.md`)
- `inbox/raw/` is the only inlet and is sacred: never edited or deleted. (`CLAUDE.md`)
- Never invent fields: what is unknown is omitted, no placeholders. (`CLAUDE.md`)
- Technical artifacts (files, comments, docs) default to English; user-facing view labels stay in Spanish as designed ("Por empresa", "Por rol", "Sin empresa", "Sin rol").
- No Dataview, no formulas beyond the empty-value fallbacks, no reconnection/classification/PARA features (out of scope — S3/S5/S6).

---

### Task 1: The Base — `people/contacts.base`

**Files:**
- Create: `people/contacts.base`
- Reference (confirm only, do not rewrite): `people/_template.md`, `people/fede-olgue.md`

**Interfaces:**
- Consumes: person-note frontmatter fields `type`, `name`, `company`, `role`, `relationship`, `last_contact` (defined in `people/_template.md`).
- Produces: a Base with two views named "Por empresa" and "Por rol" that any Obsidian 1.9+ vault renders.

- [ ] **Step 1: Confirm the template feeds the Base**

Read `people/_template.md` and verify these frontmatter keys exist and are spelled exactly: `type`, `name`, `company`, `role`, `relationship`, `last_contact`. The Base groups and orders by these. Expected: all present (per design, no change needed). If any is missing or misspelled, STOP and report before proceeding — the Base would silently show empty columns otherwise.

- [ ] **Step 2: Write `people/contacts.base`**

```yaml
formulas:
  company_group: 'if(note.company, note.company, "Sin empresa")'
  role_group: 'if(note.role, note.role, "Sin rol")'

filters:
  and:
    - file.inFolder("people")
    - 'note.type == "person"'
    - 'note.name'

views:
  - type: table
    name: "Por empresa"
    groupBy:
      property: formula.company_group
    order:
      - note.name
      - note.role
      - note.relationship
      - note.last_contact
  - type: table
    name: "Por rol"
    groupBy:
      property: formula.role_group
    order:
      - note.name
      - note.company
      - note.relationship
      - note.last_contact
```

- [ ] **Step 3: Verify the Base YAML is valid**

Run: `python3 -c "import yaml,sys; yaml.safe_load(open('people/contacts.base')); print('valid YAML')"`
Expected: prints `valid YAML` (no traceback).

- [ ] **Step 4: Verify the template-exclusion filter is sound**

Confirm by inspection that `people/_template.md` has an empty `name:` and `people/fede-olgue.md` has a non-empty `name:`.
Run: `rg -n "^name:" people/_template.md people/fede-olgue.md`
Expected: `_template.md` shows `name:` with nothing after it; `fede-olgue.md` shows `name:` followed by a value. This is what the `note.name` filter relies on to keep the template out of the views.

- [ ] **Step 5: VERIFY → STOP**

Show the created `people/contacts.base` to the user and the two verification results. Wait for explicit OK before committing (vault rule: never auto-commit).

- [ ] **Step 6: Commit (only after OK)**

```bash
git add people/contacts.base
git commit -m "feat: add contacts base with company and role views"
```

---

### Task 2: Usage doc — `people/README.md`

**Files:**
- Create: `people/README.md`

**Interfaces:**
- Consumes: the Base and views produced in Task 1 (names "Por empresa", "Por rol").
- Produces: human-readable documentation of the add-contact flow and how to read the Base.

- [ ] **Step 1: Write `people/README.md`**

```markdown
# people/ — the CRM

One note per person, `people/<slug>.md` (`<slug>` = kebab-case of the name),
built from `people/_template.md`.

## Adding a contact

Contacts are never added by hand-editing this folder. The flow is:

1. Raw text lands in `inbox/raw/` and stays **untouched** (the sacred inlet).
2. The agent extracts the fields and creates/updates `people/<slug>.md` from
   the template — dedupe by `name`/`aliases` first, never invent fields.
3. VERIFY → STOP: the agent shows the proposed note and waits for your OK
   before committing.

## Reading the directory — `contacts.base`

`contacts.base` renders the CRM as a queryable table with two views:

- **Por empresa** — contacts grouped by `company`. Missing company →
  **"Sin empresa"**.
- **Por rol** — contacts grouped by `role`. Missing role → **"Sin rol"**.

Open `contacts.base` in Obsidian (1.9+) and switch views from the tab bar.
The `_template.md` note is filtered out (it has no `name`).
```

- [ ] **Step 2: Verify the doc matches reality**

Run: `rg -n "Por empresa|Por rol|Sin empresa|Sin rol|inbox/raw" people/README.md`
Expected: all four view/group labels and the `inbox/raw` inlet reference are present, matching the Base and the vault rules.

- [ ] **Step 3: VERIFY → STOP**

Show `people/README.md` to the user. Wait for explicit OK before committing.

- [ ] **Step 4: Commit (only after OK)**

```bash
git add people/README.md
git commit -m "docs: add people/ CRM usage readme"
```

---

## Self-Review

**Spec coverage:**
- `contacts.base` with two views + fallback formulas → Task 1. ✓
- Empty values as "Sin empresa" / "Sin rol" → Task 1, Step 2 formulas. ✓
- Template excluded via `note.name` filter → Task 1, Steps 2 & 4. ✓
- Template review (confirm consistency) → Task 1, Step 1. ✓
- `people/README.md` (add-contact flow + how to read the Base) → Task 2. ✓
- TRIGGER → DO → VERIFY → STOP before commit → both tasks, VERIFY steps. ✓

**Placeholder scan:** No TBD/TODO; all file content and commands are concrete.

**Type consistency:** Frontmatter keys (`type`, `name`, `company`, `role`,
`relationship`, `last_contact`), formula names (`company_group`, `role_group`),
and view names ("Por empresa", "Por rol") are used identically across tasks and
the README.
