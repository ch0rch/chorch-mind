# people/ — the CRM

One note per person, `people/<slug>.md` (`<slug>` = kebab-case of the name),
built from `people/_template.md`.

## Adding a contact

Contacts are never added by hand-editing this folder. The flow is:

1. Raw text lands in `inbox/raw/` and stays **untouched** (the sacred inlet).
2. The agent extracts the fields and creates/updates `people/<slug>.md` from
   the template — dedupe by `name`/`aliases` first, never invent fields.
3. VERIFY → STOP: the agent shows the proposed note and waits for your OK
   before committing.

## Reading the directory — `contacts.base`

`contacts.base` renders the CRM as a queryable table with two views:

- **Por empresa** — contacts grouped by `company`. Missing company →
  **"Sin empresa"**.
- **Por rol** — contacts grouped by `role`. Missing role → **"Sin rol"**.

Open `contacts.base` in Obsidian (1.9+) and switch views from the tab bar.
The `_template.md` note is filtered out (it has no `name`).
