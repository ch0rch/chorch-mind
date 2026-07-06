# Arquitectura target — Second brain personal (borrador)

> Estado: **borrador para decisión**. Reúne el stack, la estructura del vault y los
> puntos abiertos que hay que resolver con el cliente antes de construir.

## Objetivo y límite

- **Objetivo 1 (este):** second brain personal, foco en contactos/networking.
- **Objetivo 2 (aparte, más adelante):** stack de dev AI para Balanz.
- **No se mezclan.** Balanz es una financiera → dato sensible. Límite de datos estricto.

## Stack

- **Source of truth = repo GitHub privado** = el vault Obsidian versionado con git.
  El historial git es la red de seguridad / undo ante un agente autónomo.
- **Obsidian local** (plugin Obsidian Git) = UI donde el cliente lee/edita; pull/push a GitHub.
- **Eve @ Vercel** = agente durable always-on, con canal directo (Telegram/Slack desde el celu).
  - Vercel Sandbox tiene git real: `bootstrap()` hace `git clone`, `onSession()` define network policy.
  - Credential brokering: el token de GitHub queda **fuera** del sandbox.
  - Schedules → Vercel Cron (síntesis semanal, radar de reconexión).
  - Modelo vía Vercel AI Gateway (OIDC, sin API keys sueltas).
- **Claude Code** = banco de trabajo para sesiones profundas (construir/refactorizar el vault). Clona el mismo repo.
- **GitHub es el hub:** todos los actores sincronizan por ahí.

> El sandbox de Vercel se apaga a los ~30 min de inactividad. Por eso el source of
> truth debe ser el remoto en GitHub, nunca el sandbox.

## Estructura del vault

```
/
├── CLAUDE.md              # instrucciones raíz del agente
├── inbox/
│   └── raw/               # único inlet; lo raw es sagrado (no se edita el original)
├── people/                # CRM: una nota por persona, frontmatter estructurado
│                          # vistas con Obsidian Bases / Dataview
├── daily/                 # notas diarias; ligan interacciones a personas
├── projects/             ─┐
├── areas/                 │  PARA
├── resources/             │
└── archive/              ─┘
```

## Patrón operativo

- **TRIGGER → DO → VERIFY → STOP.** Nunca auto-merge: la confirmación es siempre humana (HITL).
- **"Keys, not prompts":** permisos read-only scoped. No confiar en instrucciones para limitar al agente.
- **Manual primero, automatizar después.**
- **Split de modelos:** Haiku para clasificar/taggear, Sonnet/Opus para juicio.

## Secuencia de construcción (sesión a sesión)

| Sesión | Entregable |
|--------|------------|
| S1 | vault + `inbox/raw` + `CLAUDE.md` (entrevista inicial) |
| S2 | módulo contactos (`people/`) + Bases |
| S3 | conexión MCP + primer skill de clasificación |
| S4 | backlinks + `daily/` ligando interacciones a personas |
| S5 | PARA + wiki por tema |
| S6 | síntesis semanal + radar de reconexión |
| S7 | (graduación) desplegar Eve como capa de automatización + canal, con VERIFY/STOP y keys read-only |

## Puntos abiertos para decidir con el cliente

1. **Privacidad.** La superficie es grande: GitHub + Vercel + Telegram + Anthropic manejando
   contactos reales y datos personales. Decidir cuánto de esa exposición se acepta.
2. **Costo.** Sandbox + tokens + schedules. No es gratis; hay que dimensionarlo.
3. **Concurrencia git.** Tres actores pusheando (Obsidian, Eve, Claude Code) →
   regla **pull-before-write** + commits atómicos.

## Recomendación de fasing

Separar dos decisiones que hoy están juntas:

- **Decisión A — ¿arrancamos el vault?** Obsidian + Claude Code + GitHub privado.
  ~80% del valor con una fracción del riesgo y del costo.
- **Decisión B — ¿desplegamos el agente always-on (Eve)?** Es la graduación (S7),
  no el punto de partida. Dispara casi toda la superficie de privacidad y costo.
  Puede esperar sin bloquear el arranque.
