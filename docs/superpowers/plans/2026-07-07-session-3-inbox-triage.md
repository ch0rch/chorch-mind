# Session 3 — Inbox Triage & Portable Curation Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move contact curation into a portable skill, add an inbox-triage classifier skill, and shrink `CLAUDE.md` — all agent-agnostic so they run in Claude Code and Eve.

**Architecture:** Two portable skills under `.claude/skills/`. `contact-curation` holds the six existing curation rules moved verbatim from `CLAUDE.md`. `inbox-triage` detects each raw item's form, routes contacts to `contact-curation`, and for everything else presents a PARA hint plus destination options and lets the user choose. `CLAUDE.md` loses its curation section and points to the skills.

**Tech Stack:** Markdown SKILL.md files (YAML frontmatter + body), git.

## Global Constraints

- Source of truth is the git repo; every change is committed. (`CLAUDE.md`)
- Operating loop is TRIGGER → DO → VERIFY → STOP. Show the change and wait for OK before committing. Never auto-merge. (`CLAUDE.md`)
- `inbox/raw/` is the only inlet and is sacred: never edited or deleted. (`CLAUDE.md`)
- Never invent fields: what is unknown is omitted, no placeholders. (`CLAUDE.md`)
- `CLAUDE.md` stays minimal; operational logic lives in portable, agent-agnostic skills under `.claude/skills/`. Skills must not hardcode any harness-specific ask mechanism.
- Vault operational artifacts (SKILL.md bodies) are written in Spanish, matching the vault. Process docs (this plan, the spec) are English.
- Moving curation to a skill must preserve behavior verbatim — same six rules, no addition, no loss.

---

### Task 1: Portable curation skill — `contact-curation`

**Files:**
- Create: `.claude/skills/contact-curation/SKILL.md`
- Reference (source of the verbatim rules): `CLAUDE.md:35-49`

**Interfaces:**
- Consumes: `people/_template.md` (the mold), `people/` notes (for dedupe).
- Produces: a skill named `contact-curation` that `inbox-triage` invokes when an item is a contact.

- [ ] **Step 1: Write `.claude/skills/contact-curation/SKILL.md`**

```markdown
---
name: contact-curation
description: Use when a contact arrives in natural language or lands in inbox/raw/. Extracts fields into people/<slug>.md from people/_template.md, dedupes by name/aliases, records met_via backlinks, never invents fields, and shows the proposed note for approval before committing (VERIFY then STOP).
---

# Curación de contactos

Cuando recibo un contacto en lenguaje natural:

1. El texto crudo entra a `inbox/raw/` y queda **intacto**.
2. El agente **extrae** los campos y crea/actualiza `people/<slug>.md` copiando
   el molde de `people/_template.md`.
3. **Dedupe**: antes de crear una nota nueva, busca por `name` y `aliases` una
   persona existente. Si existe, actualiza esa nota.
4. Si menciono quién me lo presentó, se registra en `met_via` como
   backlink `[[Nombre]]`.
5. **Nunca inventar campos**: lo que no sé, se **omite** (nada de placeholders
   ni datos supuestos).
6. **VERIFY**: el agente me muestra la nota propuesta y espera mi OK antes de
   commitear. STOP.
```

- [ ] **Step 2: Verify frontmatter is valid and rules are complete**

Run: `rg -n "^name:|^description:" .claude/skills/contact-curation/SKILL.md && rg -c "^[0-9]\." .claude/skills/contact-curation/SKILL.md`
Expected: `name:` and `description:` both present; the numbered-rule count is `6`.

- [ ] **Step 3: Verify no rule was lost vs the CLAUDE.md source**

Confirm by inspection that each of the six rules in `CLAUDE.md:39-49` appears verbatim in the skill body (intacto, extrae, dedupe, met_via, nunca inventar, VERIFY/STOP).
Run: `rg -n "intacto|Dedupe|met_via|inventar|VERIFY" .claude/skills/contact-curation/SKILL.md`
Expected: all five anchor phrases present.

- [ ] **Step 4: VERIFY → STOP**

Show the new skill to the user. Wait for explicit OK before committing.

- [ ] **Step 5: Commit (only after OK)**

```bash
git add .claude/skills/contact-curation/SKILL.md
git commit -m "feat: add portable contact-curation skill"
```

---

### Task 2: The classifier skill — `inbox-triage`

**Files:**
- Create: `.claude/skills/inbox-triage/SKILL.md`

**Interfaces:**
- Consumes: raw items in `inbox/raw/`; the `contact-curation` skill (Task 1) for the contact form.
- Produces: a skill named `inbox-triage` that routes contacts automatically and asks the user where non-contacts go.

- [ ] **Step 1: Write `.claude/skills/inbox-triage/SKILL.md`**

```markdown
---
name: inbox-triage
description: Use when raw items land in inbox/raw/ and need routing. Detects each item's coarse form (contact, note/idea, task, reference); routes contacts to the contact-curation skill automatically; for everything else presents a PARA hint plus destination options and lets the user choose. raw stays sacred; VERIFY then STOP before writing.
---

# Triage del inbox

Cuando entran items a `inbox/raw/`, para cada uno:

1. Leo el item de `inbox/raw/`. El crudo es **sagrado**: no se edita ni se borra.
2. Detecto su **forma**: contacto, nota/idea, tarea o referencia.
3. Si es **contacto** → uso el skill `contact-curation`. Destino claro
   (`people/`), no hace falta preguntar.
4. Si **no es contacto**:
   a. Calculo una **pista PARA** orientativa (no vinculante): `project?`,
      `area?` o `resource?`.
   b. **Le presento al usuario**: qué detecté + la pista + opciones de destino:
      `projects/` · `areas/` · `resources/` · `inbox/triaged/` (holding, "todavía
      no sé") · descartar. (Agnóstico al agente: describí las opciones y esperá
      la elección; no asumas un mecanismo de pregunta particular.)
   c. Espero la **elección del usuario**.
   d. Creo la nota derivada en el destino elegido, con el crudo **intacto** y un
      backlink `source` al original.
5. **VERIFY → STOP**: no escribo nada hasta que el usuario elige el destino.

## Nota derivada (frontmatter)

​```yaml
form: note | task | reference
para_hint: project | area | resource
destination: <la elección del usuario>
source: "[[<nombre-del-raw-sin-extensión>]]"
triaged: <YYYY-MM-DD>
​```

Debajo del frontmatter, un resumen/extracto del item. **Nunca inventar campos.**
```

- [ ] **Step 2: Verify frontmatter and agent-agnostic wording**

Run: `rg -n "^name:|^description:" .claude/skills/inbox-triage/SKILL.md && rg -n "Agnóstico al agente|contact-curation|VERIFY" .claude/skills/inbox-triage/SKILL.md`
Expected: `name:`/`description:` present; the agent-agnostic note, the `contact-curation` delegation, and the VERIFY step all present.

- [ ] **Step 3: VERIFY → STOP**

Show the new skill to the user. Wait for explicit OK before committing.

- [ ] **Step 4: Commit (only after OK)**

```bash
git add .claude/skills/inbox-triage/SKILL.md
git commit -m "feat: add inbox-triage classifier skill"
```

---

### Task 3: Shrink `CLAUDE.md` and add the staging folder

**Files:**
- Modify: `CLAUDE.md:35-49` (remove the curation section, add a pointer)
- Create: `inbox/triaged/.gitkeep`

**Interfaces:**
- Consumes: the two skills from Tasks 1-2 (they must exist first — this task removes the rules they now hold).
- Produces: a minimal `CLAUDE.md` that points to `.claude/skills/`, and the `inbox/triaged/` staging folder.

- [ ] **Step 1: Replace the curation section in `CLAUDE.md`**

Delete lines 35-49 (the whole `## Reglas de curación de contactos` section) and replace with:

```markdown
## Lógica operativa: skills portables

La lógica operativa del vault (curación de contactos, triage del inbox) vive en
**skills portables** en `.claude/skills/`, agnósticas al agente para correr
igual en Claude Code y en Eve:

- `contact-curation` — un contacto en lenguaje natural → nota en `people/`.
- `inbox-triage` — clasifica lo que entra a `inbox/raw/` y lo rutea.

Este `CLAUDE.md` se mantiene chico: estructura, loop y principios. La lógica,
en los skills.
```

- [ ] **Step 2: Create the staging folder**

Run: `touch inbox/triaged/.gitkeep`
Expected: file exists.

- [ ] **Step 3: Verify the shrink and pointer**

Run: `rg -n "Reglas de curación de contactos" CLAUDE.md; rg -n "skills portables|contact-curation|inbox-triage" CLAUDE.md`
Expected: the old section header is **gone** (no match on the first); the pointer and both skill names are present (matches on the second).

- [ ] **Step 4: Verify no behavior lost — rules now live in the skill**

Run: `rg -c "^[0-9]\." .claude/skills/contact-curation/SKILL.md`
Expected: `6` — the six rules removed from `CLAUDE.md` are preserved in the skill.

- [ ] **Step 5: VERIFY → STOP**

Show the shrunk `CLAUDE.md` diff and the new folder to the user. Wait for explicit OK before committing.

- [ ] **Step 6: Commit (only after OK)**

```bash
git add CLAUDE.md inbox/triaged/.gitkeep
git commit -m "refactor: move curation rules to skills, shrink CLAUDE.md"
```

---

## Self-Review

**Spec coverage:**
- `contact-curation` skill, rules moved verbatim → Task 1. ✓
- `inbox-triage` skill, 4 forms, contact auto-routes, non-contact asks with
  hint + options, agent-agnostic → Task 2. ✓
- Derived-note frontmatter (`form`, `para_hint`, `destination`, `source`,
  `triaged`) → Task 2, Step 1. ✓
- `CLAUDE.md` shrunk + pointer → Task 3, Step 1. ✓
- `inbox/triaged/` staging folder → Task 3, Step 2. ✓
- Behavior preserved (six rules, verbatim) → Task 1 Step 3 + Task 3 Step 4. ✓
- TRIGGER → DO → VERIFY → STOP before every commit → all tasks. ✓
- Task ordering: skills created (1,2) BEFORE the rules are removed from
  `CLAUDE.md` (3) — no window where behavior is lost. ✓

**Placeholder scan:** No TBD/TODO; every file's content and command is concrete.

**Type consistency:** Skill names (`contact-curation`, `inbox-triage`) and
frontmatter field names (`form`, `para_hint`, `destination`, `source`,
`triaged`) are used identically across the spec, both skills, and the pointer.
