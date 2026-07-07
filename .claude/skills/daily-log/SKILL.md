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

   ```markdown
   ---
   type: daily
   date: <YYYY-MM-DD>
   ---

   ## Interacciones

   ## Notas del día
   ```
3. **Actualizo `last_contact`** de esa persona a la fecha de hoy.
4. **Nunca inventar campos**: lo que no sé, se omite.
5. **VERIFY → STOP**: muestro el cambio (línea en el daily + `last_contact`
   actualizado) y espero mi OK antes de commitear.

El crudo de `inbox/raw/` es sagrado; la nota diaria y la de la persona son
derivadas y editables. (Agnóstico al agente: describí el cambio y esperá el OK;
no asumas un mecanismo de confirmación particular.)
