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

   ```markdown
   ---
   type: review
   period: <semana YYYY-MM-DD>
   date: <YYYY-MM-DD>
   ---

   ## Interacciones de la semana

   ## Personas contactadas

   ## Notas creadas / archivadas

   ## Hilos abiertos
   ```
4. **Nunca inventar actividad** que no esté en las notas.
5. **VERIFY → STOP**: muestro el digest y espero el OK antes de commitear.

(Agnóstico al agente: describí el digest y esperá el OK; no asumas un mecanismo
de confirmación particular.)
