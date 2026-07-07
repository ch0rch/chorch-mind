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
