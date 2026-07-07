# chorch-mind — un second brain que vos controlás

> Documento de presentación conceptual. Explica cómo funciona el stack hoy y
> cómo se suma Eve. Pensado para leer o compartir.

---

## La idea en una frase

**Tu conocimiento y tu red profesional, versionados en git como fuente de
verdad, operados por agentes de IA que proponen y nunca deciden solos.**

No es una app. No es una base de datos en la nube de otro. Es un repositorio
que vos controlás, donde la IA trabaja *para vos* dentro de límites que vos
ponés.

---

## Por qué así — los principios

Estos seis principios son el corazón del diseño. Todo lo demás se deriva de acá.

1. **El repo es la fuente de verdad.** Todo vive versionado en git. El historial
   es tu botón de "deshacer" ante cualquier acción de un agente. Nada se pierde,
   todo es reversible.
2. **Keys, not prompts.** La seguridad la dan los *permisos* (read-only, con
   alcance acotado), no la confianza en que el prompt se porte bien. No le pedís
   por favor al agente que no rompa nada: no le das la llave para hacerlo.
3. **TRIGGER → DO → VERIFY → STOP.** El agente *propone* un cambio y *espera tu
   OK*. Nunca hace auto-merge. El humano siempre lidera; la IA ejecuta.
4. **Manual primero, automatizar después.** Cada capacidad se prueba a mano
   antes de dejarla correr sola.
5. **PARA — organizar por accionabilidad, no por tema.** Las notas fluyen entre
   categorías según qué tan cerca están de una acción. Nada tiene un lugar fijo
   para siempre.
6. **El inbox es sagrado.** Lo que entra crudo no se edita ni se borra. Es la
   materia prima; el agente *deriva* de ella, nunca la altera.

---

## El harness — cómo está armado el repo

El repositorio *es* el producto. Su estructura es simple y legible a propósito:

```
chorch-mind/
├── CLAUDE.md         # las reglas que el agente lee antes de tocar nada (chico)
├── inbox/raw/        # el ÚNICO inlet — lo crudo es sagrado
├── people/           # el CRM — una nota por persona
├── daily/            # notas diarias — ligan interacciones a personas
├── projects/         ┐
├── areas/            │  PARA — por accionabilidad, no por tema
├── resources/        │
├── archive/          ┘
├── reviews/          # síntesis semanales
└── .claude/skills/   # LA LÓGICA — skills portables, agnósticas al agente
```

Dos archivos merecen atención especial:

- **`CLAUDE.md`** son las reglas del vault. Se mantiene deliberadamente **chico**:
  estructura, el loop operativo y los principios. Nada más.
- **`.claude/skills/`** es donde vive toda la inteligencia operativa.

---

## Las skills — la lógica portable

**Este es el concepto más importante para entender hacia dónde va todo.**

La inteligencia del sistema NO está atada a Claude Code. Está encapsulada en
**skills**: instrucciones portables y *agnósticas al agente*. Cada skill
describe *qué* hacer y *qué* confirmar con el usuario — nunca *cómo* preguntarlo.

¿Por qué importa? Porque el mismo skill corre igual en Claude Code (hoy, desde
la laptop) y en Eve (mañana, desde Telegram). **No hay que reescribir nada para
migrar.** La disciplina de mantener la lógica portable es lo que hace que sumar
Eve sea trivial en vez de un rework.

Los seis skills que ya funcionan:

| Skill | Qué hace |
|-------|----------|
| `contact-curation` | Un contacto en lenguaje natural → nota estructurada en `people/`. |
| `inbox-triage` | Clasifica lo que entra al inbox y lo rutea (contacto, nota, tarea, referencia). |
| `daily-log` | Registra una interacción con una persona en `daily/` con backlink, y actualiza `last_contact`. |
| `para-classify` | Clasifica una nota en PARA por accionabilidad; contempla que las notas *fluyen*. |
| `reconnect-radar` | *(solo lectura)* sugiere a quién reconectar según hace cuánto no lo contactás. |
| `weekly-synthesis` | Digest semanal de la actividad → nota en `reviews/`. |

---

## El CRM — cómo funciona

El módulo de contactos es el uso principal hoy. Funciona así:

- **Una nota por persona**, con frontmatter estructurado (nombre, empresa, rol,
  cómo lo conociste, último contacto, canales…). El schema está pensado
  *DB-ready*: si mañana querés migrar a una base de datos, el mapeo es directo.
- **Entrada conversacional.** No cargás formularios. Le contás en lenguaje
  natural ("conocí a X, fundador de Y, hablamos de Z") y el agente *extrae* los
  campos, *deduplica* contra la gente que ya tenés, y *linkea* quién te presentó
  a quién.
- **Vistas con Obsidian Bases.** El CRM es *consultable*: agrupá contactos por
  empresa o por rol con una vista declarativa, sin plugins.
- **Interacciones vía backlinks.** Cuando registrás una charla en la nota del
  día con `[[Persona]]`, la nota de esa persona *muestra la interacción sola*
  (backlinks nativos de Obsidian). Se escribe una vez, se ve en los dos lados.
- **`last_contact` siempre fresco** → alimenta el radar de reconexión: el
  sistema sabe a quién estás dejando enfriar.

---

## El loop operativo completo

Todo input recorre el mismo ciclo de vida:

```
capturar → triar → curar → registrar → clasificar → reconectar → sintetizar
  inbox     triage   people    daily       PARA         radar        reviews
```

Cada paso es un skill. El humano confirma en cada punto donde algo se escribe.

---

## Hoy y mañana — los dos modos de uso

El stack está pensado para **dos superficies de trabajo** que comparten el mismo
repo y los mismos skills:

### Hoy — Claude Code desde la laptop

- **Banco de trabajo y constructor principal.** Acá se construye y refactoriza
  el vault, se hacen las sesiones profundas.
- Clona el mismo repo de GitHub. Trabajo denso, con foco.

### Mañana — Eve desde Telegram

- **Capa always-on.** Eve es un agente durable que vive en Vercel, siempre
  disponible.
- **Canal natural desde el celular:** le hablás por Telegram y captura contactos
  o interacciones al vuelo, en el momento en que pasan.
- **Schedules automáticos:** la síntesis semanal y el radar de reconexión corren
  solos (por cron), y te llegan al chat.
- **Mismos skills, mismas reglas.** Keys read-only, VERIFY → STOP: Eve también
  propone y espera tu OK. No es un agente suelto.

**GitHub es el hub** que sincroniza todo: laptop, Eve y tu Obsidian local
empujan y tiran del mismo repo.

---

## Estado actual y roadmap

| Sesión | Entregable | Estado |
|--------|------------|--------|
| S1 | Cimientos del vault + `CLAUDE.md` | ✅ |
| S2 | Módulo de contactos + Bases | ✅ |
| S3 | Triage del inbox (primer clasificador) | ✅ |
| S4 | `daily/` + interacciones con backlinks | ✅ |
| S5 | Clasificación PARA + semilla de wiki | ✅ |
| S6 | Síntesis semanal + radar de reconexión | ✅ |
| S7 | **Desplegar Eve** (canal + automatización) | ⏳ la graduación |

---

## El concepto que lo ata todo

Separamos **dos decisiones** que suelen ir juntas y no deberían:

- **Decisión A — ¿arrancamos el vault?** Obsidian + Claude Code + GitHub
  privado. Es ~80% del valor con una fracción del riesgo y del costo. *Ya está
  hecho.*
- **Decisión B — ¿desplegamos el agente always-on (Eve)?** Es la graduación. Se
  suma *cuando el vault ya funciona*, no como punto de partida.

Construimos primero el cerebro. Eve es el sistema nervioso que lo conecta al
mundo — y llega cuando el cerebro ya está sano.
