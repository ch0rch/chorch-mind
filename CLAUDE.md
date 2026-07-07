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

## Lógica operativa: skills portables

La lógica operativa del vault (curación de contactos, triage del inbox) vive en
**skills portables** en `.claude/skills/`, agnósticas al agente para correr
igual en Claude Code y en Eve:

- `contact-curation` — un contacto en lenguaje natural → nota en `people/`.
- `inbox-triage` — clasifica lo que entra a `inbox/raw/` y lo rutea.
- `daily-log` — registra interacciones con personas en `daily/` (backlinks).
- `para-classify` — clasifica notas en PARA (`projects/ areas/ resources/ archive/`) por accionabilidad.
- `reconnect-radar` — sugiere a quién reconectar (read-only) por `last_contact`.
- `weekly-synthesis` — digest semanal de `daily/` → nota en `reviews/`.

Las notas-tema (MOC, "wiki por tema") viven en `resources/` a partir de
`resources/_topic-template.md`.

Este `CLAUDE.md` se mantiene chico: estructura, loop y principios. La lógica,
en los skills.
