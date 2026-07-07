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
