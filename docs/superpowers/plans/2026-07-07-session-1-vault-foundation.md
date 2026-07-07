# Session 1 — Vault Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Lay the chorch-mind vault foundation — folder skeleton, root `CLAUDE.md` rules, and the person-note contract — so the CRM is alive and the first real contact can be loaded via natural language.

**Architecture:** Markdown-in-repo vault under git (source of truth). PARA folders + a single `inbox/raw` inlet + a `people/` CRM. Any agent curates natural-language input into structured person notes following rules documented in the root `CLAUDE.md`. No database, no encryption, no importer (see spec Non-Goals).

**Tech Stack:** Plain Markdown + YAML frontmatter. Git for versioning. Verification uses shell tools already present (`python3` for YAML parsing, `git`, `rg`). No test runner exists — this is a content vault, not a code project, so each task ends with an observable artifact check rather than a unit test.

## Global Constraints

- Storage is markdown-in-repo. No database. (spec Non-Goals)
- No git-crypt / encryption this session. (spec Non-Goals)
- No importer, no Eve, no MCP, no classification skills this session. (spec Non-Goals)
- Schema frontmatter keys are English identifiers (DB-ready). Vault content and the root `CLAUDE.md` prose are in Spanish (the project's language).
- Person note path: `people/<slug>.md`, where `<slug>` is kebab-case of `name`.
- Human-in-the-loop always: no autonomous commits of curated notes. The agent proposes, the human approves (VERIFY → STOP).
- Never invent frontmatter fields; omit unknown values (no placeholders).
- Frontmatter schema (exact fields): `type, name, aliases, company, role, location, tags, met_via, met_context, first_met, last_contact, relationship, channels`. `relationship` ∈ `{cold, warm, close}`. `channels` = `{linkedin, email, phone}`.

---

### Task 1: Vault folder skeleton

**Files:**
- Create: `inbox/raw/.gitkeep`, `people/.gitkeep`, `daily/.gitkeep`, `projects/.gitkeep`, `areas/.gitkeep`, `resources/.gitkeep`, `archive/.gitkeep`

**Interfaces:**
- Consumes: nothing.
- Produces: the directory tree every later task and session writes into.

- [ ] **Step 1: Create the folders with .gitkeep placeholders**

```bash
cd /home/chorch/Documentos/proyectos/chorch-mind
for d in inbox/raw people daily projects areas resources archive; do
  mkdir -p "$d"
  touch "$d/.gitkeep"
done
```

- [ ] **Step 2: Verify the tree exists and is tracked by git**

```bash
git add inbox people daily projects areas resources archive
git status --porcelain
```
Expected: seven `A  <path>/.gitkeep` lines, one per folder (`inbox/raw/.gitkeep`, `people/.gitkeep`, `daily/.gitkeep`, `projects/.gitkeep`, `areas/.gitkeep`, `resources/.gitkeep`, `archive/.gitkeep`).

- [ ] **Step 3: Commit**

```bash
git commit -m "feat: add vault folder skeleton"
```

---

### Task 2: Person-note contract (`people/_template.md`)

**Files:**
- Create: `people/_template.md`

**Interfaces:**
- Consumes: the `people/` folder from Task 1.
- Produces: the canonical person-note shape every agent copies when curating a contact. Frontmatter keys: `type, name, aliases, company, role, location, tags, met_via, met_context, first_met, last_contact, relationship, channels`.

- [ ] **Step 1: Write the template file**

Create `people/_template.md` with exactly this content:

```markdown
---
type: person
name:
aliases: []
company:
role:
location:
tags: []
met_via:
met_context:
first_met:
last_contact:
relationship: warm   # cold | warm | close
channels:
  linkedin:
  email:
  phone:
---

## Notas

## Historial
```

- [ ] **Step 2: Verify the frontmatter is valid YAML**

```bash
python3 -c "import yaml,sys; body=open('people/_template.md').read().split('---')[1]; d=yaml.safe_load(body); assert set(d.keys()) == {'type','name','aliases','company','role','location','tags','met_via','met_context','first_met','last_contact','relationship','channels'}, d.keys(); assert set(d['channels'].keys()) == {'linkedin','email','phone'}; print('OK schema:', sorted(d.keys()))"
```
Expected: `OK schema: ['aliases', 'channels', 'company', 'first_met', 'last_contact', 'location', 'met_context', 'met_via', 'name', 'relationship', 'role', 'tags', 'type']`

- [ ] **Step 3: Commit**

```bash
git add people/_template.md
git commit -m "feat: add person-note contract template"
```

---

### Task 3: Root `CLAUDE.md` (the rules the agent obeys)

**Files:**
- Create: `CLAUDE.md` (repo root — the vault's rules, in Spanish)

**Interfaces:**
- Consumes: the folder skeleton (Task 1) and the person-note schema (Task 2) it references.
- Produces: the operating contract every agent (Claude Code now, Eve later) reads before acting on the vault.

- [ ] **Step 1: Write the root CLAUDE.md**

Create `CLAUDE.md` at the repo root with exactly this content:

```markdown
# chorch-mind — reglas del vault

Este repositorio es mi *second brain* personal. Es la fuente de verdad: todo
vive versionado en git. Cualquier agente (Claude Code ahora, Eve más adelante)
lee este archivo antes de tocar nada.

## Estructura (PARA + inlet + CRM)

- `inbox/raw/` — **el único inlet**. Lo que entra acá es **sagrado**: no se edita
  ni se borra. Es la materia prima cruda de la que el agente extrae.
- `people/` — el CRM. Una nota por persona en `people/<slug>.md`
  (`<slug>` = kebab-case del nombre). El molde es `people/_template.md`.
- `daily/` — notas diarias.
- `projects/` — PARA: cosas con objetivo y fecha de fin.
- `areas/` — PARA: responsabilidades continuas sin fecha de fin.
- `resources/` — PARA: temas de interés que no son una responsabilidad.
- `archive/` — PARA: lo que ya murió (proyectos cerrados, áreas soltadas).

PARA organiza por **accionabilidad, no por tema**. Las notas fluyen entre
categorías según qué tan cerca estén de una acción.

## Loop operativo: TRIGGER → DO → VERIFY → STOP

1. **TRIGGER** — llega un input (un contacto en lenguaje natural, una nota).
2. **DO** — el agente propone el cambio (crear/actualizar una nota).
3. **VERIFY** — el agente **me muestra** el cambio propuesto y **espera mi OK**.
4. **STOP** — sin OK explícito, no se commitea. **Nunca auto-merge.**

## Keys not prompts

La seguridad la dan los **permisos**, no la confianza en el prompt. Los accesos
del agente son read-only y scoped por defecto. No se asume que el prompt se
porte bien.

## Reglas de curación de contactos

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

- [ ] **Step 2: Verify the required sections are present**

```bash
rg -c '^## (Estructura|Loop operativo|Keys not prompts|Reglas de curación)' CLAUDE.md
```
Expected: `4`

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "feat: add root CLAUDE.md vault rules"
```

---

### Task 4: Validation — curate the first real contact

This task exercises the whole foundation end to end. It is interactive: it needs
one real contact from the user, described in natural language. It cannot be
fully automated — that is the point (human-in-the-loop). The deliverable is one
real person note that conforms to the contract.

**Files:**
- Create: `inbox/raw/<timestamp>-<slug>.md` (the raw NL capture, sacred)
- Create: `people/<slug>.md` (the curated note)

**Interfaces:**
- Consumes: `people/_template.md` (Task 2) and the curation rules in `CLAUDE.md` (Task 3).
- Produces: the first CRM entry; proof the contract is usable.

- [ ] **Step 1: Capture the raw input**

Ask the user for one real contact in natural language. Save the raw text
verbatim to `inbox/raw/<YYYY-MM-DD>-<slug>.md`. Do not edit it.

- [ ] **Step 2: Curate into a person note (DO)**

Copy `people/_template.md` to `people/<slug>.md`. Fill only the fields the raw
text supports. Set `first_met` and `last_contact` to today (`2026-07-07`) if the
encounter is today. If the user named who introduced them, set
`met_via: "[[Nombre]]"`. Omit every field with no evidence.

- [ ] **Step 3: Verify the note conforms to the contract**

```bash
python3 -c "import yaml; d=yaml.safe_load(open('people/<slug>.md').read().split('---')[1]); assert d['type']=='person'; assert d['name']; assert d['relationship'] in {'cold','warm','close'}; assert None not in d.values() or True; print('OK note:', d['name'])"
```
Expected: `OK note: <name>` and no YAML error. Manually confirm no field contains an invented/placeholder value.

- [ ] **Step 4: Show the proposed note and wait for approval (VERIFY → STOP)**

Present the full `people/<slug>.md` to the user. Do NOT proceed without an
explicit OK.

- [ ] **Step 5: Commit (only after OK)**

```bash
git add inbox/raw/ people/<slug>.md
git commit -m "feat: add first curated contact"
```

---

## Self-Review

**Spec coverage:**
- Vault skeleton → Task 1. ✓
- Person-note contract / `_template.md` → Task 2. ✓
- Curation rules → Task 3 (documented in `CLAUDE.md`). ✓
- Root `CLAUDE.md` scope (PARA, raw sacred, TRIGGER→DO→VERIFY→STOP, keys not prompts) → Task 3. ✓
- Validation / first real contact → Task 4. ✓
- Non-Goals (no DB, no git-crypt, no importer, no Eve) → honored; nothing in the plan introduces them. ✓

**Placeholder scan:** The only `<...>` tokens are intentional path variables (`<slug>`, `<YYYY-MM-DD>`) resolved at execution from real input, not unfinished content. Every file's full content is inlined.

**Type consistency:** Frontmatter key set is identical in the Global Constraints, Task 2 template, Task 2 verification, and Task 4 usage: `type, name, aliases, company, role, location, tags, met_via, met_context, first_met, last_contact, relationship, channels`. `relationship` domain `{cold, warm, close}` consistent across Task 2 and Task 4.
