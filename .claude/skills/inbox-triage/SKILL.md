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

```yaml
form: note | task | reference
para_hint: project | area | resource
destination: <la elección del usuario>
source: "[[<nombre-del-raw-sin-extensión>]]"
triaged: <YYYY-MM-DD>
```

Debajo del frontmatter, un resumen/extracto del item. **Nunca inventar campos.**
