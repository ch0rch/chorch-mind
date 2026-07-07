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
