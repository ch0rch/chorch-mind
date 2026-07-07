# Session 6 — Weekly Synthesis & Reconnection Radar Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add two portable periodic-review skills — `reconnect-radar` (read-only) and `weekly-synthesis` — plus a `reviews/` folder, and point `CLAUDE.md` at them.

**Architecture:** `reconnect-radar` scans `people/` and flags contacts overdue by `relationship`-based cadence using existing `last_contact`. `weekly-synthesis` digests the last 7 days of `daily/` into a saved review note under `reviews/`. Both are agent-agnostic; the radar never mutates the vault.

**Tech Stack:** Markdown (YAML frontmatter + body) SKILL.md and vault notes, git.

## Global Constraints

- Source of truth is the git repo; every change is committed. (`CLAUDE.md`)
- Skills preserve TRIGGER → DO → VERIFY → STOP behavior even though this build session runs autonomously (user-authorized for S6).
- `inbox/raw/` is sacred; review notes are derived. (`CLAUDE.md`)
- Never invent activity/fields not present in the notes. (`CLAUDE.md`)
- `CLAUDE.md` stays minimal; logic lives in portable, agent-agnostic skills. No hardcoded mechanism.
- Vault operational artifacts (SKILL.md bodies) are Spanish; process docs are English.
- Radar cadence thresholds: close > 14d, warm > 45d, cold > 90d (orientation).

---

### Task 1: Read-only reconnection radar — `reconnect-radar`

**Files:**
- Create: `.claude/skills/reconnect-radar/SKILL.md`

**Interfaces:**
- Consumes: `people/*.md` frontmatter (`last_contact`, `relationship`, `name`, `channels`).
- Produces: a read-only skill that lists people due for reconnection.

- [ ] **Step 1: Write `.claude/skills/reconnect-radar/SKILL.md`**

```markdown
---
name: reconnect-radar
description: Use to see which people are due for reconnection. Read-only: scans people/ notes, computes days since last_contact, and flags contacts past a cadence threshold based on relationship (close > 14d, warm > 45d, cold > 90d). Suggests who to reach out to; never mutates the vault.
---

# Radar de reconexión

Cuando me pregunto **a quién debería reconectar** (manual, o agendado bajo Eve):

1. **Leo las notas de `people/`.** Para cada persona, calculo los días desde
   `last_contact` hasta hoy.
2. **Marco a la persona** si el hueco supera la cadencia de su `relationship`
   (orientación, no ley):
   - `close` — reconectar si pasaron **más de 14 días**.
   - `warm` — reconectar si pasaron **más de 45 días**.
   - `cold` — reconectar si pasaron **más de 90 días**.
3. **Presento la lista** ordenada por cuán atrasada está cada reconexión, con
   nombre, días desde el último contacto y canal disponible.
4. **Read-only**: solo sugiero, **nunca escribo ni muevo nada** en el vault.

(Agnóstico al agente: en Claude Code muestro la lista; en Eve va por el canal.
No asumas un mecanismo de salida particular.)
```

- [ ] **Step 2: Verify frontmatter, thresholds, and read-only guard**

Run: `rg -n "^name:|^description:" .claude/skills/reconnect-radar/SKILL.md && rg -n "14 días|45 días|90 días|nunca escribo|Read-only" .claude/skills/reconnect-radar/SKILL.md`
Expected: `name:`/`description:` present; all three thresholds and the read-only guard present.

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/reconnect-radar/SKILL.md
git commit -m "feat: add read-only reconnect-radar skill"
```

---

### Task 2: Weekly synthesis — `weekly-synthesis` + `reviews/`

**Files:**
- Create: `.claude/skills/weekly-synthesis/SKILL.md`
- Create: `reviews/.gitkeep`

**Interfaces:**
- Consumes: last 7 days of `daily/` notes; recent CRM movement.
- Produces: a skill that writes a weekly digest to `reviews/<today>.md`, and the `reviews/` folder.

- [ ] **Step 1: Write `.claude/skills/weekly-synthesis/SKILL.md`**

```markdown
---
name: weekly-synthesis
description: Use to produce a weekly review. Reads the last 7 days of daily/ notes and CRM movement, digests interactions, people contacted, notes created/filed, and open threads, and writes a review note to reviews/<today>.md. Shows the digest and waits for approval before committing (VERIFY then STOP).
---

# Síntesis semanal

Cuando hago la **revisión semanal** (manual, o agendada bajo Eve):

1. **Leo los últimos 7 días de `daily/`** y el movimiento reciente del CRM.
2. **Armo el digest**: interacciones registradas (con sus links `[[Persona]]`),
   personas contactadas, notas creadas/archivadas en PARA, e hilos abiertos.
3. **Escribo el digest** en `reviews/<hoy>.md` con esta estructura:

   ​```markdown
   ---
   type: review
   period: <semana YYYY-MM-DD>
   date: <YYYY-MM-DD>
   ---

   ## Interacciones de la semana

   ## Personas contactadas

   ## Notas creadas / archivadas

   ## Hilos abiertos
   ​```
4. **Nunca inventar actividad** que no esté en las notas.
5. **VERIFY → STOP**: muestro el digest y espero el OK antes de commitear.

(Agnóstico al agente: describí el digest y esperá el OK; no asumas un mecanismo
de confirmación particular.)
```

- [ ] **Step 2: Create the reviews folder**

Run: `mkdir -p reviews && touch reviews/.gitkeep`
Expected: `reviews/.gitkeep` exists.

- [ ] **Step 3: Verify frontmatter, structure, and VERIFY step**

Run: `rg -n "^name:|^description:" .claude/skills/weekly-synthesis/SKILL.md && rg -n "type: review|Hilos abiertos|VERIFY" .claude/skills/weekly-synthesis/SKILL.md && test -f reviews/.gitkeep && echo "reviews/ ok"`
Expected: `name:`/`description:` present; the review structure and VERIFY step present; `reviews/` folder tracked.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/weekly-synthesis/SKILL.md reviews/.gitkeep
git commit -m "feat: add weekly-synthesis skill and reviews folder"
```

---

### Task 3: Point `CLAUDE.md` at the two new skills

**Files:**
- Modify: `CLAUDE.md` (extend the skills list)

**Interfaces:**
- Consumes: the skills from Tasks 1-2 (they must exist first).
- Produces: an updated, still-minimal pointer.

- [ ] **Step 1: Extend the skills list in `CLAUDE.md`**

After the `para-classify` bullet, add:

```markdown
- `reconnect-radar` — sugiere a quién reconectar (read-only) por `last_contact`.
- `weekly-synthesis` — digest semanal de `daily/` → nota en `reviews/`.
```

- [ ] **Step 2: Verify the pointer lists both new skills**

Run: `rg -n "reconnect-radar|weekly-synthesis" CLAUDE.md`
Expected: both new skills are referenced.

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: point CLAUDE.md at reconnect-radar and weekly-synthesis"
```

---

## Self-Review

**Spec coverage:**
- `reconnect-radar` read-only, three cadence thresholds, ordered output → Task 1. ✓
- `weekly-synthesis` reads 7 days, digest structure, writes to reviews/,
  VERIFY→STOP → Task 2. ✓
- `reviews/` folder separate from daily/ → Task 2 Step 2. ✓
- `CLAUDE.md` pointer updated → Task 3. ✓
- Radar uses existing last_contact/relationship, no new fields → Task 1. ✓
- Skills preserve TRIGGER→DO→VERIFY→STOP (synthesis writes; radar read-only) →
  Tasks 1-2. ✓

**Placeholder scan:** No TBD/TODO; every file's content and command is concrete.

**Type consistency:** Skill names (`reconnect-radar`, `weekly-synthesis`),
frontmatter keys (`type`, `period`, `date`), thresholds (14/45/90d), and section
headers are used identically across the spec, skills, and pointer.
