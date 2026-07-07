# Session 5 — PARA Classification & Topic Wiki Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a portable `para-classify` skill, seed the topic wiki with a MOC template, and point `CLAUDE.md` at both.

**Architecture:** `para-classify` systematizes the PARA hints from S3 into explicit actionability criteria (projects/areas/resources/archive), consuming `inbox-triage`'s `para_hint` as orientation. A MOC template seeds the topic wiki. `CLAUDE.md` stays minimal with an extended pointer.

**Tech Stack:** Markdown (YAML frontmatter + body) SKILL.md and vault notes, git.

## Global Constraints

- Source of truth is the git repo; every change is committed. (`CLAUDE.md`)
- Skills preserve TRIGGER → DO → VERIFY → STOP behavior even though this build session runs autonomously (user-authorized for S5 only).
- `inbox/raw/` is sacred; classified notes are derived/editable. (`CLAUDE.md`)
- Never invent fields. (`CLAUDE.md`)
- `CLAUDE.md` stays minimal; logic lives in portable, agent-agnostic skills. No hardcoded ask mechanism.
- Vault operational artifacts (SKILL.md bodies, templates) are Spanish; process docs are English.
- PARA criteria are taken verbatim from `CLAUDE.md`: projects = objective + end date; areas = ongoing responsibility, no end date; resources = interest, not a responsibility; archive = dead.

---

### Task 1: Portable PARA classifier — `para-classify`

**Files:**
- Create: `.claude/skills/para-classify/SKILL.md`

**Interfaces:**
- Consumes: notes in `inbox/triaged/` (with `form`/`para_hint` from S3) or existing notes; the PARA folders `projects/ areas/ resources/ archive/`.
- Produces: a skill named `para-classify` that proposes and (on approval) moves a note to a PARA folder.

- [ ] **Step 1: Write `.claude/skills/para-classify/SKILL.md`**

```markdown
---
name: para-classify
description: Use when filing a note into PARA (projects/areas/resources/archive) or re-classifying one whose actionability changed. Classifies by actionability using the PARA criteria, consuming any para_hint from inbox-triage as orientation, proposes a destination with a reason, and moves the note on approval. raw stays sacred; VERIFY then STOP.
---

# Clasificación PARA

Cuando hay que archivar una nota en PARA (o reclasificar una cuya
accionabilidad cambió):

1. **TRIGGER**: una nota a clasificar — típicamente de `inbox/triaged/` (trae
   `form` y `para_hint` del triage) o una nota existente a reevaluar.
2. **Aplico el criterio PARA por accionabilidad** (no por tema):
   - `projects/` — tiene un **objetivo y una fecha de fin**.
   - `areas/` — una **responsabilidad continua** sin fecha de fin.
   - `resources/` — un **tema de interés** que no es una responsabilidad.
   - `archive/` — lo que ya **murió** (proyecto cerrado, área soltada).
   Uso el `para_hint` del triage como **orientación, no como veredicto**.
3. **Propongo el destino con una razón de una línea.** Como las notas
   **fluyen**, esto puede significar mover una nota ya archivada en otra
   categoría.
4. Con el OK, **muevo la nota** a la carpeta PARA elegida (conservo sus
   backlinks; el crudo de `inbox/raw/` queda **intacto**). Nunca inventar campos.
5. **VERIFY → STOP**: muestro el destino propuesto y la razón, y espero el OK
   antes de mover nada.

(Agnóstico al agente: describí la propuesta y esperá el OK; no asumas un
mecanismo de confirmación particular.)
```

- [ ] **Step 2: Verify frontmatter and the four criteria + flow behavior**

Run: `rg -n "^name:|^description:" .claude/skills/para-classify/SKILL.md && rg -n "projects/|areas/|resources/|archive/|fluyen|VERIFY" .claude/skills/para-classify/SKILL.md`
Expected: `name:`/`description:` present; all four PARA folders, the "fluyen" (re-classify) behavior, and the VERIFY step present.

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/para-classify/SKILL.md
git commit -m "feat: add portable para-classify skill"
```

---

### Task 2: Topic wiki seed — MOC template

**Files:**
- Create: `resources/_topic-template.md`

**Interfaces:**
- Produces: a topic-note (MOC) template used to group related notes by topic via backlinks.

- [ ] **Step 1: Write `resources/_topic-template.md`**

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

- [ ] **Step 2: Verify the template**

Run: `rg -n "type: topic|Notas relacionadas|Referencias" resources/_topic-template.md`
Expected: the `type: topic` frontmatter and the "Notas relacionadas" and "Referencias" sections are present.

- [ ] **Step 3: Commit**

```bash
git add resources/_topic-template.md
git commit -m "feat: add topic MOC template as wiki-by-topic seed"
```

---

### Task 3: Point `CLAUDE.md` at the new skill and topic MOC

**Files:**
- Modify: `CLAUDE.md` (extend the "Lógica operativa: skills portables" list)

**Interfaces:**
- Consumes: the skill and template from Tasks 1-2 (they must exist first).
- Produces: an updated, still-minimal pointer.

- [ ] **Step 1: Extend the skills list in `CLAUDE.md`**

After the `inbox-triage` bullet, add:

```markdown
- `daily-log` — registra interacciones con personas en `daily/` (backlinks).
- `para-classify` — clasifica notas en PARA (`projects/ areas/ resources/ archive/`) por accionabilidad.

Las notas-tema (MOC, "wiki por tema") viven en `resources/` a partir de
`resources/_topic-template.md`.
```

(Note: `daily-log` was added in S4 but not yet listed; include it here so the
pointer is complete.)

- [ ] **Step 2: Verify the pointer lists all skills and the MOC**

Run: `rg -n "contact-curation|inbox-triage|daily-log|para-classify|_topic-template" CLAUDE.md`
Expected: all four skills and the topic template are referenced.

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: point CLAUDE.md at para-classify and topic MOC"
```

---

## Self-Review

**Spec coverage:**
- `para-classify` skill, four PARA criteria, hint-as-orientation, notes-flow
  re-classification, VERIFY→STOP → Task 1. ✓
- Topic MOC template (wiki seed) → Task 2. ✓
- `CLAUDE.md` pointer updated (incl. the previously-unlisted `daily-log`) →
  Task 3. ✓
- Skills preserve TRIGGER→DO→VERIFY→STOP → Task 1 Step 1. ✓

**Placeholder scan:** No TBD/TODO; every file's content and command is concrete.

**Type consistency:** Skill name (`para-classify`), PARA folder names, frontmatter
keys (`type`, `title`, `tags`), and section headers (`Notas relacionadas`,
`Referencias`) are used identically across the spec, skill, and template.
