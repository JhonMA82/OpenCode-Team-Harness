<p align="center">
  <img src="harness_banner.png" alt="Banner de Harness" width="600">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Fork-Puerto_OpenCode-orange.svg" alt="Bifurcación">
  <img src="https://img.shields.io/badge/Cambio-Agentes→Tareas_Paralelas-red.svg" alt="Cambio de arquitectura">
</p>

> ⚠️ **Port a OpenCode**: Versión adaptada a OpenCode de
> [revfactory/harness](https://github.com/JhonMA82/OpenCode-Team-Harness).
>
> - **Arquitectura**: Equipos de agentes → tareas paralelas
> - **Compatibilidad**: Solo OpenCode (NO compatible con Claude Code)
> - Ver [CHANGES.md](CHANGES.md) para detalles completos.

<p align="center">
  <img src="https://img.shields.io/badge/Version-1.0.1-brightgreen.svg" alt="Versión">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="Licencia"></a>
  <img src="https://img.shields.io/badge/OpenCode-Skill-blue.svg" alt="Habilidad de OpenCode">
  <img src="https://img.shields.io/badge/Patterns-6_Architectures-orange.svg" alt="6 Patrones de Arquitectura">
  <img src="https://img.shields.io/badge/Modo-Tareas_Paralelas-orange.svg" alt="Tareas en paralelo">
  <a href="https://github.com/JhonMA82/OpenCode-Team-Harness/stargazers"><img src="https://img.shields.io/github/stars/revfactory/harness?style=social" alt="Estrellas en GitHub"></a>
</p>

# Harness

**Arquitecto de Equipos y Skills** — Skill de OpenCode

Un meta-skill que diseña equipos de agentes especializados por dominio, define
agentes y genera las skills que utilizan.

## Visión General

Harness aprovecha el sistema de tareas en paralelo de OpenCode para descomponer
tareas complejas en equipos coordinados de agentes especializados. Con solo
decir "construí un harness para este proyecto", genera automáticamente
definiciones de agentes (`.opencode/agents/`) y skills
(`.opencode/skills/`) adaptadas a tu dominio.

## Características Principales

- **Diseño de Equipos** — 6 patrones arquitectónicos: canalización,
  distribución/reunificación, grupo de expertos, productor-revisor, supervisor
  y delegación jerárquica
- **Generación de Skills** — Genera skills automáticas con Divulgación
  Progresiva para una gestión eficiente del contexto
- **Orquestación** — Paso de datos entre agentes, manejo de errores y
  protocolos de coordinación
- **Validación** — Verificación de disparadores, pruebas en seco y tests
  comparativos con/sin skill

## Flujo de Trabajo

```
Fase 1: Análisis de Dominio
    ↓
Fase 2: Diseño de Arquitectura del Equipo (equipos de agentes vs. subagentes)
    ↓
Fase 3: Generación de Definiciones de Agentes (.opencode/agents/)
    ↓
Fase 4: Generación de Skills (.opencode/skills/)
    ↓
Fase 5: Integración y Orquestación
    ↓
Fase 6: Validación y Pruebas
```

## Instalación

### Instalación local por proyecto

Instalá Harness dentro del proyecto donde querés usarlo. Esta es la instalación
recomendada porque mantiene la configuración versionada con el repositorio y
evita modificar la configuración global de OpenCode.

```shell
# Desde el proyecto consumidor
mkdir -p .opencode/skills
cp -r /ruta/a/OpenCode-Team-Harness/skills/harness .opencode/skills/harness
```

## Estructura del proyecto

```
harness/
├── skills/
│   └── harness/
│       ├── SKILL.md                # Definición principal de la skill (flujo de 6 fases)
│       └── references/
│           ├── agent-design-patterns.md   # 6 patrones arquitectónicos
│           ├── orchestrator-template.md   # Plantillas de orquestación
│           ├── team-examples.md           # 5 configuraciones de equipos reales
│           ├── skill-writing-guide.md     # Guía de autoría de skills
│           ├── skill-testing-guide.md     # Metodología de pruebas y evaluación
│           └── qa-agent-guide.md          # Guía de integración del agente QA
└── README.md
```

## Uso

Actívalo en OpenCode con frases como:

```
Construye un harness para este proyecto
Diseña un equipo de agentes para este dominio
Configura un harness
```

### Modos de ejecución

| Modo | Descripción | Recomendado Para |
| --- | --- | --- |
| **Tareas en paralelo** (por defecto) | Herramienta de tareas con `run_in_background: true` | 2+ agentes que requieren colaboración |
| **Tareas secuenciales** | Herramienta de tareas con ejecución ordenada | Tareas únicas, sin comunicación entre agentes |

<p align="center">
  <img src="harness_team.png" alt="Equipo de Agentes Harness" width="500">
</p>

### Patrones arquitectónicos

| Patrón | Descripción |
| --- | --- |
| Canalización | Tareas secuenciales dependientes |
| Distribución/reunificación | Tareas paralelas independientes |
| Grupo de expertos | Invocación selectiva según contexto |
| Productor-Revisor | Generación seguida de revisión de calidad |
| Supervisor | Agente central con distribución dinámica de tareas |
| Delegación Jerárquica | Delegación recursiva de arriba a abajo |

## Salida

Archivos generados por Harness:

```
tu-proyecto/
├── .opencode/
│   ├── agents/          # Archivos de definición de agentes
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── qa.md
│   └── skills/          # Archivos de skills
│       ├── analyze/
│       │   ├── SKILL.md
│       │   └── references/
│       └── build/
│           ├── SKILL.md
│           └── references/
```

## Casos de Uso — Probá Estos Prompts

Copiá cualquier prompt en OpenCode después de instalar Harness:

**Investigación Profunda**

```
Construí un harness para investigación profunda. Necesito un equipo de agentes
capaz de investigar cualquier tema desde múltiples ángulos — búsqueda web,
fuentes académicas, opinión de la comunidad — para luego validar hallazgos
y producir un informe completo.
```

**Desarrollo de sitios web**

```
Construí un harness para desarrollo integral de sitios web. El equipo debe
manejar diseño, interfaz (React/Next.js), servidor (API) y QA en un flujo
coordinado desde el wireframe hasta el despliegue.
```

**Producción de Webtoons / Cómics**

```
Construí un harness para producción de episodios de webtoon. Necesito agentes
para guion, diseño de personajes, planificación de viñetas y edición de
diálogos. Deben revisar el trabajo de los demás para mantener consistencia
de estilo.
```

**Planificación de Contenido para YouTube**

```
Construí un harness para creación de contenido en YouTube. El equipo debe
investigar tendencias, escribir guiones, optimizar títulos/etiquetas para
buscadores y planificar conceptos de miniaturas — todo coordinado por un agente
supervisor.
```

**Revisión y Refactorización de Código**

```
Construí un harness para revisión integral de código. Quiero agentes en
paralelo revisando arquitectura, vulnerabilidades de seguridad, cuellos de
botella de rendimiento y estilo de código — para luego fusionar todo en un
solo informe.
```

**Documentación Técnica**

```
Construí un harness que genere documentación de API a partir de este código.
Los agentes deben analizar endpoints, escribir descripciones, generar ejemplos
de uso y revisar que esté completa.
```

**Diseño de Pipelines de Datos**

```
Construí un harness para diseñar pipelines de datos. Necesito agentes para
diseño de esquemas, lógica ETL, reglas de validación y configuración de
monitoreo que deleguen subtareas jerárquicamente.
```

**Campaña de Marketing**

```
Construí un harness para creación de campañas de marketing. El equipo debe
investigar el mercado objetivo, redactar copys publicitarios, diseñar conceptos
visuales y definir planes de pruebas A/B con revisión iterativa de calidad.
```

## Construido con Harness

### Harness 100

**[revfactory/harness-100](https://github.com/revfactory/harness-100)** — 100
harnesses de equipos de agentes listos para producción en 10 dominios,
disponibles en inglés y coreano (200 paquetes en total). Cada harness incluye
4-5 agentes especializados, una skill de orquestación y skills específicas del
dominio — todo generado por este proyecto. 1.808 archivos Markdown que cubren
creación de contenido, desarrollo de software, datos/IA, estrategia de negocio,
educación, legal, salud y más.

### Investigación: prueba A/B de efectividad de Harness

**[revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)**
— Un experimento controlado en 15 tareas de ingeniería de software que mide el
impacto de la preconfiguración estructurada en la calidad de la salida de
agentes LLM.

| Métrica | Sin Harness | Con Harness | Mejora |
| --- | :-: | :-: | :-: |
| Puntaje de Calidad Promedio | 49.5 | 79.3 | **+60%** |
| Tasa de Éxito | — | — | **100%** (15/15) |
| Varianza de Salida | — | — | **-32%** |

Hallazgo clave: la efectividad escala con la complejidad de la tarea — cuanto
más difícil, mayor la mejora (+23.8 Básico, +29.6 Avanzado, +36.2 Experto).

> Artículo completo: *Hwang, M. (2026). Harness: Structured Pre-Configuration
> for Enhancing LLM Code Agent Output Quality.*

## Requisitos

- OpenCode con soporte de skills habilitado

## Licencia

Apache 2.0
