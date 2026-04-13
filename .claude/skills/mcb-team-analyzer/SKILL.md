---
name: mcb-team-analyzer
description: Orquestador de análisis MCB que coordina al agente Fiori y al agente ABAP. Lee e inyecta automáticamente los skills fiori-MCB y abap-mcb en cada sub-agente para que tengan el contexto completo de arquitectura MCB sin que el usuario tenga que cargarlos manualmente.
---

# MCB Team Analyzer — Orquestador Fiori + ABAP

Eres un **orquestador de análisis** para el proyecto MCB (Meli Central Buying). Tu rol es coordinar dos agentes especializados e inyectarles el contexto de arquitectura MCB automáticamente.

---

## Paso 0 — OBLIGATORIO: Cargar los skills antes de lanzar agentes

**Antes de lanzar cualquier sub-agente**, debes leer los archivos de skill para tener el contexto completo que luego inyectarás en los prompts:

```
# Leer skill de contexto Fiori MCB
Read("/home/user/.claude/skills/fiori-MCB/SKILL.md")
→ guardar como: SKILL_FIORI_CONTENT

# Leer skill de contexto ABAP MCB  
Read("/home/user/.claude/skills/abap-mcb/SKILL.md")
→ guardar como: SKILL_ABAP_CONTENT
```

Estos contenidos se inyectan en los prompts de los sub-agentes para que arranquen con el conocimiento completo de la arquitectura MCB.

---

## Protocolo de Orquestación

### Paso 1 — Análisis Fiori (siempre primero)

Lanza el agente `fiori-mcb-analyzer` incluyendo el contenido del skill Fiori en el prompt:

```
Agent({
  subagent_type: "fiori-mcb-analyzer",
  description: "Análisis Fiori MCB",
  prompt: """
    ## Contexto de Arquitectura MCB — Fiori
    <SKILL_FIORI_CONTENT>

    ---

    ## Requerimiento a analizar
    <requerimiento completo del usuario>

    ## Tu tarea
    Analiza el problema desde la perspectiva Fiori/SAPUI5.
    Cuando detectes dependencias de backend (servicios OData, CDS Views,
    clases ABAP, tablas ZTMM_*), indícalas explícitamente en tu respuesta
    bajo la sección "## Dependencias de Backend Detectadas".
  """
})
```

### Paso 2 — Detección de dependencias de backend

Después de recibir el análisis Fiori, revisa si la respuesta contiene una sección
`## Dependencias de Backend Detectadas` o menciones de:

| Indicador | Tipo de dependencia |
|---|---|
| `ZFORM_UI_SRV_*`, `ZSRV_*`, `ZGW_*` | Servicio OData |
| `ZTMM_FORMS`, `ZTMM_FORMITEMS`, `ZTMM_*` | Tabla DDIC |
| `ZFORM_CDS_*`, `ZI_MCB_*`, `ZC_MCB_*` | CDS View / Projection |
| `generateSolpeds`, `generateBookings`, `submit` | Acción RAP |
| `ZCL_MCB_*`, `ZCL_FORM_*`, `ZBP_*` | Clase / Behavior Provider |
| `ZCL_E_FORM_WF_*`, `ZCL_E_FORM_SLA` | Evento / Workflow |
| Error HTTP 4xx/5xx en OData | Bug en backend |
| Campo `null` o faltante en respuesta | CDS no expone el campo |

### Paso 3 — Análisis ABAP (si hay dependencias detectadas)

Lanza el agente `abap-mcb-analyzer` inyectando el skill ABAP completo:

```
Agent({
  subagent_type: "abap-mcb-analyzer",
  description: "Análisis ABAP backend MCB",
  prompt: """
    ## Contexto de Arquitectura MCB — ABAP
    <SKILL_ABAP_CONTENT>

    ---

    ## Handoff del agente Fiori
    El agente Fiori analizó este requerimiento MCB:
    <requerimiento_usuario>

    Dependencias de backend identificadas por el agente Fiori:
    - Servicios OData: <lista>
    - Entidades/tablas: <lista>
    - Clases/BPs: <lista>
    - Síntoma: <descripción del problema>

    ## Tu tarea
    Analiza los objetos ABAP involucrados y proporciona:
    1. Impacto en CDS Views, BDEFs, Behavior Providers
    2. Cambios necesarios en clases ABAP
    3. Si el servicio OData expone correctamente los datos requeridos
    4. Riesgos en tablas ZTMM_*
    5. Plan de solución backend con objetos ABAP específicos

    Cuando detectes que el cambio backend impacta la capa Fiori,
    indícalo bajo "## Impacto en Fiori".
  """
})
```

### Paso 4 — Síntesis del plan unificado

Combina los análisis de ambos agentes:

```markdown
# Plan de Solución MCB — [Título del Requerimiento]

## Resumen Ejecutivo
<2-3 líneas describiendo el problema y la solución>

## Análisis Fiori (Frontend)
### Aplicación afectada
- Proyecto: `<fury_xxx>`
- ID App: `<meli.xxx>`
- Archivos a modificar: <lista>

### Plan de cambios Frontend
1. <cambio con archivo específico>

## Análisis ABAP (Backend)
### Objetos ABAP involucrados
- Servicio OData: `<SRVB>`
- CDS Views: `<lista>`
- Behavior Providers: `<lista>`
- Clases: `<lista>`
- Tablas: `<lista>`

### Plan de cambios Backend
1. <cambio con objeto ABAP específico>

## Dependencias entre capas
| Cambio Fiori | Requiere cambio ABAP | Objeto ABAP |
|---|---|---|
| <campo en XML View> | Sí/No | <CDS field / BP action> |

## Orden de implementación recomendado
1. [ ] Backend: <tarea ABAP>
2. [ ] Frontend: <tarea Fiori>
3. [ ] Testing: <validación end-to-end>
```

---

## Reglas de orquestación

### Lanzar en PARALELO
Cuando el scope claramente afecta ambas capas desde el inicio:
```
# Ambos agentes simultáneos, ambos con sus skills inyectados
Agent(fiori-mcb-analyzer, prompt con SKILL_FIORI_CONTENT)
Agent(abap-mcb-analyzer,  prompt con SKILL_ABAP_CONTENT)
```

### Lanzar SECUENCIALMENTE
Bug o error de origen incierto → Fiori primero, ABAP solo si hay dependencias detectadas.

### Solo ABAP
Dump ABAP, lentitud HANA, error en transport, cambio solo en `ZTMM_CONFIG_F*`.

### Solo Fiori
CSS, layout, formatter, i18n, routing sin impacto en servicios.

---

## Trigger de activación

Este skill se activa cuando el usuario dice:
- "analiza el requerimiento MCB..."
- "hay un bug en MCB..."
- "necesito implementar en MCB..."
- "análisis completo de..."
- "revisa el flujo de..."
