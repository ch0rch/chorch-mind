---
name: contact-curation
description: Use when a contact arrives in natural language or lands in inbox/raw/. Extracts fields into people/<slug>.md from people/_template.md, dedupes by name/aliases, records met_via backlinks, never invents fields, and shows the proposed note for approval before committing (VERIFY then STOP).
---

# Curación de contactos

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
