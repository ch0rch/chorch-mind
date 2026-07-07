# Session 4 — Daily Notes & Interaction Backlinks Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `daily/` the source of truth for interactions via a portable `daily-log` skill, then migrate the existing Historial data out of person notes and retire the section.

**Architecture:** One portable skill `daily-log` appends interactions to `daily/<today>.md` with `[[Person]]` backlinks and updates the person's `last_contact`. Obsidian's linked mentions surface interactions in person notes, so the manual `## Historial` is removed. Existing Historial data is migrated to `daily/` before removal so nothing is lost.

**Tech Stack:** Markdown (YAML frontmatter + body) SKILL.md and vault notes, git.

## Global Constraints

- Source of truth is the git repo; every change is committed. (`CLAUDE.md`)
- Operating loop is TRIGGER → DO → VERIFY → STOP. Show the change and wait for OK before committing. Never auto-merge. (`CLAUDE.md`)
- `inbox/raw/` is the only inlet and is sacred; person and daily notes are derived/editable. (`CLAUDE.md`)
- Never invent fields; what is unknown is omitted. (`CLAUDE.md`)
- `CLAUDE.md` stays minimal; operational logic lives in portable, agent-agnostic skills. Skills must not hardcode any harness-specific ask mechanism.
- Vault operational artifacts (SKILL.md bodies, notes) are Spanish; process docs (this plan, the spec) are English.
- Migration must not lose data: the daily note holding the migrated interaction is created BEFORE the `## Historial` section is removed.

---

### Task 1: Portable interaction skill — `daily-log`

**Files:**
- Create: `.claude/skills/daily-log/SKILL.md`

**Interfaces:**
- Consumes: `people/` notes (match a person by `name`/`aliases`); the daily-note structure defined in this task.
- Produces: a skill named `daily-log` that logs an interaction to `daily/<today>.md` and updates `last_contact`.

- [ ] **Step 1: Write `.claude/skills/daily-log/SKILL.md`**

```markdown
---
name: daily-log
description: Use when logging an interaction with a person (a meeting, call, message). Appends the interaction to daily/<today>.md with a [[Person]] backlink, matching the person in people/ by name/aliases, and updates that person's last_contact. Creates the day note if missing. Shows the change and waits for approval before committing (VERIFY then STOP).
---

# Registro de interacciones (daily)

Cuando registro una interacción con una persona en lenguaje natural
(un meet, una llamada, un mensaje):

1. **Matcheo la persona** en `people/` por `name`/`aliases` (mismo dedupe que
   la curación). Si no hay match, lo digo — **no invento una persona**.
2. **Agrego una línea** en `daily/<hoy>.md`, bajo `## Interacciones`, que
   empieza con `[[Nombre de la persona]]` seguido de un resumen corto. Si la
   nota del día no existe, la creo con esta estructura:

   ​```markdown
   ---
   type: daily
   date: <YYYY-MM-DD>
   ---

   ## Interacciones

   ## Notas del día
   ​```
3. **Actualizo `last_contact`** de esa persona a la fecha de hoy.
4. **Nunca inventar campos**: lo que no sé, se omite.
5. **VERIFY → STOP**: muestro el cambio (línea en el daily + `last_contact`
   actualizado) y espero mi OK antes de commitear.

El crudo de `inbox/raw/` es sagrado; la nota diaria y la de la persona son
derivadas y editables. (Agnóstico al agente: describí el cambio y esperá el OK;
no asumas un mecanismo de confirmación particular.)
```

- [ ] **Step 2: Verify frontmatter and key wording**

Run: `rg -n "^name:|^description:" .claude/skills/daily-log/SKILL.md && rg -n "last_contact|Interacciones|no invento|VERIFY" .claude/skills/daily-log/SKILL.md`
Expected: `name:`/`description:` present; the `last_contact` update, the `## Interacciones` target, the no-invent guard, and the VERIFY step all present.

- [ ] **Step 3: VERIFY → STOP**

Show the new skill to the user. Wait for explicit OK before committing.

- [ ] **Step 4: Commit (only after OK)**

```bash
git add .claude/skills/daily-log/SKILL.md
git commit -m "feat: add portable daily-log interaction skill"
```

---

### Task 2: Migrate Historial to `daily/` and retire the section

**Files:**
- Create: `daily/2026-07-07.md`
- Modify: `people/fede-olgue.md` (remove `## Historial` section)
- Modify: `people/_template.md` (remove `## Historial` section)

**Interfaces:**
- Consumes: the daily-note structure from Task 1; the existing Historial entry in `people/fede-olgue.md`.
- Produces: `daily/2026-07-07.md` holding the migrated interaction; person notes and template without `## Historial`.

- [ ] **Step 1: Create `daily/2026-07-07.md` with the migrated interaction (do this FIRST — before any removal)**

```markdown
---
type: daily
date: 2026-07-07
---

## Interacciones

- [[Fede Olgue]] — Nos conectamos vía X. Primera charla por Meet.

## Notas del día
```

- [ ] **Step 2: Verify the migrated data exists**

Run: `rg -n "\[\[Fede Olgue\]\]|type: daily" daily/2026-07-07.md`
Expected: both the `type: daily` frontmatter and the `[[Fede Olgue]]` interaction are present. The Historial data is now safely in `daily/`.

- [ ] **Step 3: Remove `## Historial` from `people/fede-olgue.md`**

Delete these trailing lines from `people/fede-olgue.md`:

```markdown

## Historial

- 2026-07-07 — Nos conectamos vía X. Primera charla por Meet.
```

Leave the note ending at the `## Notas` section content.

- [ ] **Step 4: Remove `## Historial` from `people/_template.md`**

Delete the trailing `## Historial` heading, leaving the template ending at the `## Notas` section.

- [ ] **Step 5: Verify the section is retired and no data lost**

Run: `rg -n "## Historial" people/_template.md people/fede-olgue.md; echo "exit: $?"`
Expected: no matches (exit 1) — the section is gone from both.
Run: `rg -n "Nos conectamos vía X" daily/2026-07-07.md`
Expected: match — the interaction text survives in the daily note.

- [ ] **Step 6: Verify last_contact preserved**

Run: `rg -n "^last_contact:" people/fede-olgue.md`
Expected: `last_contact: 2026-07-07` (unchanged).

- [ ] **Step 7: VERIFY → STOP**

Show the new `daily/2026-07-07.md`, the trimmed `fede-olgue.md`, and the trimmed template to the user. Wait for explicit OK before committing.

- [ ] **Step 8: Commit (only after OK)**

```bash
git add daily/2026-07-07.md people/fede-olgue.md people/_template.md
git commit -m "refactor: migrate Historial to daily notes, retire the section"
```

---

## Self-Review

**Spec coverage:**
- `daily-log` portable skill, matches person, appends to daily, updates
  last_contact, agent-agnostic → Task 1. ✓
- Daily-note structure (`type: daily`, `## Interacciones`, `## Notas del día`)
  → Task 1 Step 1. ✓
- Migrate Fede's Historial to `daily/2026-07-07.md` → Task 2 Steps 1-2. ✓
- Retire `## Historial` from template and fede-olgue → Task 2 Steps 3-5. ✓
- last_contact preserved (2026-07-07) → Task 2 Step 6. ✓
- No data lost: daily created BEFORE removal → Task 2 ordering (Step 1 before
  Steps 3-4). ✓
- TRIGGER → DO → VERIFY → STOP before every commit → both tasks. ✓

**Placeholder scan:** No TBD/TODO; every file's content and command is concrete.

**Type consistency:** Skill name (`daily-log`), frontmatter keys (`type`,
`date`, `last_contact`), and section headers (`## Interacciones`, `## Notas del
día`) are used identically across the spec, the skill, and the daily note.
