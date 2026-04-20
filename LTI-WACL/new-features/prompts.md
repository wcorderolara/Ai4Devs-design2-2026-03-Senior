# 1. Brainstorming nuevas funcionalidades

## Rol y Contexto
Eres un Product Designer Senior con especialización en sistemas HR Tech y
experiencia facilitando sesiones de Design Thinking para productos SaaS B2B.
Has trabajado en equipos de producto que han llevado funcionalidades de IA
y automatización a plataformas de reclutamiento como Greenhouse, Lever y
Workday. Conoces los pain points reales de recruiters, hiring managers y
equipos de HR en empresas medianas.

## Tarea
Facilita una sesión de brainstorming estructurada para diseñar las siguientes
funcionalidades dentro del sistema ATS ya definido:

1. Eficiencia para departamentos de HR
2. Colaboración en tiempo real entre reclutadores y managers
3. Automatizaciones
4. Asistencia de IA en diversas tareas

## Contexto del sistema
- Sistema: ATS MVP SaaS B2B multi-tenant
- Usuarios principales: Recruiter, HiringManager, Admin, Interviewer
- Stack: Angular, Node.js/Express, PostgreSQL, AWS
- Modelo de datos existente: [PEGAR MODELO DE DATOS]
- Casos de uso definidos: [PEGAR CASOS DE USO]

## Estructura del brainstorming
Para cada una de las 4 funcionalidades genera las siguientes secciones:

### 1. Definición del problema
- Pain point actual que resuelve esta funcionalidad
- A quién afecta (actor/rol)
- Impacto si no se resuelve

### 2. How Might We
- 3 a 5 preguntas "How Might We" que enmarquen el espacio de solución
- Formuladas desde la perspectiva del usuario afectado

### 3. Ideas generadas (divergencia)
- Mínimo 6 ideas por funcionalidad, sin filtrar por viabilidad técnica
- Clasificadas en: Quick Win / Mediano Plazo / Innovación

### 4. Ideas priorizadas (convergencia)
- Top 3 ideas seleccionadas con criterio: impacto alto + esfuerzo razonable
- Para cada idea: nombre, descripción de una oración, actor beneficiado

### 5. Casos de uso candidatos
- Lista de 2 a 4 nuevos casos de uso que se derivarían de implementar
  las ideas priorizadas
- Formato: verbo + objeto (ej. "Sugerir candidatos automáticamente")
- Actor responsable y módulo al que pertenecería en el sistema existente

### 6. Impacto en el modelo de datos
- Entidades existentes que se verían modificadas
- Nuevas entidades o campos que podrían ser necesarios
- Clasificación: cambio menor / cambio mayor / entidad nueva

## Instrucciones de formato
- Usa H2 para cada funcionalidad principal
- Usa H3 para cada subsección del brainstorming
- Usa tablas para la priorización de ideas y el impacto en el modelo
- El output debe ser Markdown válido y en español
- Agrega un resumen ejecutivo al final con las 5 ideas más importantes
  del brainstorming completo, independientemente de la funcionalidad

## Restricciones
- Las ideas deben ser coherentes con el stack y modelo de datos ya definidos
- No propongas integraciones externas que estén fuera del alcance MVP
  (no SSO, no videollamadas, no job boards en esta fase)
- Las ideas de IA deben ser realizables con LLMs via API (OpenAI, Anthropic)
  sin requerir modelos propios entrenados
- Cada caso de uso candidato debe poder trazarse a al menos una entidad
  del modelo de datos existente

## Audiencia
El output será usado por el equipo de producto para decidir qué
funcionalidades entran en el siguiente ciclo de desarrollo del ATS.
Debe ser lo suficientemente concreto para estimar esfuerzo y lo
suficientemente creativo para no limitarse a lo obvio.

## Criterio de éxito
Al terminar el brainstorming, el equipo debe poder responder:
¿cuáles son las 3 funcionalidades de mayor impacto para implementar
primero, y qué cambios requieren en el sistema existente?



# 2. Casos de Uso - Nuevas funcionalidades
## Rol y Contexto
Eres un Analista de Sistemas Senior y Product Owner con más de 10 años
de experiencia definiendo casos de uso para sistemas SaaS B2B en el
dominio HR Tech. Conoces los estándares UML para especificación de casos
de uso y has trabajado con equipos ágiles traduciendo necesidades de
negocio en requerimientos funcionales precisos para plataformas de
reclutamiento como Greenhouse, Lever y Workday.

## Tarea
Define los casos de uso detallados para las siguientes funcionalidades
del sistema ATS, basándote en los resultados del brainstorming previo
y en los casos de uso ya existentes en el sistema:

1. Eficiencia para departamentos de HR
2. Colaboración en tiempo real entre reclutadores y managers
3. Automatizaciones
4. Asistencia de IA en diversas tareas

## Contexto del sistema
- Sistema: ATS MVP SaaS B2B multi-tenant
- Actores existentes: Recruiter, HiringManager, Admin, Interviewer, Candidate
- Casos de uso existentes: [PEGAR CASOS DE USO DEL SISTEMA]
- Modelo de datos: [PEGAR MODELO DE DATOS]
- Resultados del brainstorming: [PEGAR RESULTADOS DEL BRAINSTORMING]

## Estructura por caso de uso
Para cada caso de uso genera las siguientes secciones:

**Identificador:** formato UC-XX donde XX es número secuencial
continuando desde el último UC existente en el sistema

**Nombre:** verbo + objeto en infinitivo (ej. "Sugerir candidatos por IA")

**Módulo:** módulo del sistema al que pertenece el caso de uso

**Actor principal:** quién inicia el caso de uso

**Actores secundarios:** quién participa pero no inicia (incluir Sistema/IA
donde aplique)

**Precondiciones:** estado del sistema antes de ejecutar el caso de uso

**Flujo principal:** pasos numerados del flujo feliz, en lenguaje de negocio,
sin términos técnicos de implementación

**Flujos alternativos:** mínimo 1 por caso de uso, indicando en qué paso
del flujo principal se bifurca

**Postcondiciones:** estado del sistema tras la ejecución exitosa

**Reglas de negocio:** restricciones e invariantes que aplican a este caso de uso

**Impacto en modelo de datos:** entidades afectadas, campos nuevos o
modificados, entidades nuevas si aplica

**Prioridad:** Alta / Media / Baja con justificación de una oración

**Dependencias:** IDs de otros casos de uso que deben existir previamente

## Agrupación por módulo
Organiza todos los casos de uso en los siguientes módulos,
añadiendo módulos nuevos si las funcionalidades lo requieren:

- Módulo existente: Gestión de Vacantes
- Módulo existente: Gestión de Candidatos
- Módulo existente: Pipeline y Seguimiento
- Módulo existente: Evaluación
- Módulo existente: Comunicación
- Módulo existente: Administración
- Módulo nuevo sugerido: Automatización
- Módulo nuevo sugerido: Asistencia IA
- Módulo nuevo sugerido: Colaboración

## Instrucciones de formato
- Usa H2 para cada módulo
- Usa H3 para cada caso de uso con su identificador y nombre
- Usa negritas para los labels de cada campo (Actor principal, Precondiciones, etc.)
- Representa las relaciones entre casos de uso al final de cada módulo
  en un bloque Mermaid con sintaxis flowchart LR
- El output debe ser Markdown válido y en español
- Al final del documento incluye una matriz de trazabilidad en tabla con:
  UC ID | Nombre | Módulo | Actor | Prioridad | Funcionalidad origen

## Restricciones
- No duplicar casos de uso ya existentes en el sistema; si una nueva
  funcionalidad extiende uno existente, usar relación extend con el UC ID original
- Los flujos deben estar escritos desde la perspectiva del actor,
  no de la implementación técnica
- Las reglas de negocio deben ser consistentes con las del modelo de datos
- Los casos de uso de IA deben describir el comportamiento esperado
  sin asumir una implementación específica de modelo o proveedor
- Máximo 10 casos de uso por módulo para mantener cohesión

## Audiencia
El output será revisado por el Tech Lead, el Product Owner y los
desarrolladores del equipo para estimar esfuerzo e iniciar el backlog
del siguiente ciclo de desarrollo. Debe ser suficientemente preciso
para escribir criterios de aceptación sin necesidad de reuniones
adicionales de refinamiento.

## Criterio de éxito
Cada caso de uso debe ser lo suficientemente completo para que un
desarrollador pueda iniciar su implementación sin ambigüedad, y lo
suficientemente claro para que el Product Owner pueda validar que
resuelve el problema de negocio identificado en el brainstorming.


# 3. Data-Model nuevas funcionalidades
## Rol y Contexto
Eres un Arquitecto de Software Senior especializado en modelado de datos
para plataformas SaaS B2B HR Tech. Has diseñado extensiones de dominio
sobre sistemas existentes sin romper contratos entre entidades, aplicando
principios de Domain-Driven Design, Open/Closed Principle y patrones de
extensibilidad como Event Sourcing y polimorfismo explícito. Conoces los
patrones de datos requeridos para features de colaboración en tiempo real,
automatizaciones basadas en triggers y asistencia con LLMs.

## Tarea
Extiende el modelo de datos existente del ATS para soportar las siguientes
4 nuevas funcionalidades, sin modificar las reglas de negocio ni los
contratos de las entidades ya definidas salvo que sea estrictamente necesario:

1. Eficiencia para departamentos de HR
2. Colaboración en tiempo real entre reclutadores y managers
3. Automatizaciones
4. Asistencia de IA en diversas tareas

## Modelo de datos existente
[MODELO DE DATOS ACTUAL — YA PROVISTO EN EL CONTEXTO]

## Casos de uso nuevos
[PEGAR CASOS DE USO GENERADOS PARA LAS NUEVAS FUNCIONALIDADES]

## Instrucciones de formato
Para cada entidad nueva o modificada proporciona:

1. **Nombre** de la entidad (PascalCase, singular)
2. **Tipo**: Nueva entidad / Extensión de entidad existente / Value Object nuevo
3. **Justificación** de una oración: por qué esta entidad es necesaria
   para soportar la funcionalidad indicada
4. **Atributos** en tabla con columnas: Atributo | Tipo | Notas
5. **Relaciones** con entidades existentes y nuevas (cardinalidad)
6. **Reglas de negocio** específicas de esta entidad (1 a 4 reglas)
7. **Clasificación DDD**: Aggregate Root / Entity / Value Object
8. **Funcionalidad que habilita**: referencia a cuál de las 4 funcionalidades
   justifica esta entidad

Para entidades existentes que requieran modificación:
- Lista solo los campos nuevos o modificados en una tabla separada
- Señala con ⚠️ cada campo nuevo o modificado
- Documenta el motivo del cambio en una línea
- Indica si el cambio es retrocompatible o requiere migración

## Estructura del documento
Organiza el output en las siguientes secciones en este orden:

### Resumen de cambios
Tabla con columnas:
Entidad | Tipo de cambio | Funcionalidad | Impacto en modelo existente

### Extensiones a entidades existentes
Solo campos nuevos agregados a entidades ya definidas,
agrupados por entidad

### Nuevas entidades por módulo funcional

#### Módulo: Eficiencia HR
Entidades que soporten dashboards, métricas, reportes y
optimización de flujos internos

#### Módulo: Colaboración en tiempo real
Entidades que soporten menciones, notificaciones, actividad
compartida y visibilidad entre reclutadores y managers

#### Módulo: Automatizaciones
Entidades que soporten triggers, reglas de automatización,
acciones programadas y condiciones de disparo

#### Módulo: Asistencia IA
Entidades que soporten sugerencias de IA, historial de
interacciones con LLM, scoring automatizado y contexto
de prompts por entidad del dominio

### Value Objects nuevos
Solo si aplican patrones de encapsulamiento de lógica de formato
o validación que se repitan en múltiples entidades nuevas

### Aggregate Roots nuevos o promovidos
Justificación de por qué alguna entidad nueva debe ser raíz
de su propio agregado

### Decisiones de modelado clave
Mínimo una decisión documentada por módulo funcional, con:
- Alternativas consideradas
- Decisión tomada
- Razón técnica o de negocio

### Impacto en modelo existente
Tabla de compatibilidad con columnas:
Entidad existente | Tipo de impacto | Descripción del cambio | Requiere migración

## Restricciones
- No modificar PKs, FKs ni tipos de atributos existentes
- No eliminar entidades ni atributos ya definidos
- No romper las reglas de negocio documentadas en el modelo actual
- Las entidades de IA no deben asumir un proveedor específico
  (usar campos genéricos: `provider`, `modelVersion`, `promptTokens`)
- Las entidades de automatización deben ser configurables por tenant
  sin hardcodear reglas en el esquema
- Las entidades de colaboración deben respetar el modelo multi-tenant:
  ninguna entidad nueva puede existir fuera de una `Company`
- Máximo 3 niveles de anidamiento en relaciones nuevas para
  mantener queries eficientes

## Audiencia
El output será revisado por el Tech Lead y el Product Owner para
validar el dominio extendido antes de generar el DDL y los diagramas
ERD actualizados. Debe ser suficientemente preciso para que el equipo
de backend inicie la implementación sin reuniones adicionales.

## Criterio de éxito
El modelo extendido debe poder responder las siguientes preguntas
sin joins excesivos ni lógica fuera del esquema:

1. ¿Qué automatizaciones se han disparado para una vacante y con qué resultado?
2. ¿Qué sugerencias de IA se han generado para un candidato y cuáles fueron aceptadas?
3. ¿Quién ha visto y comentado una postulación en tiempo real y cuándo?
4. ¿Cuáles son las métricas de eficiencia del proceso de reclutamiento
   por vacante, por reclutador y por departamento?

# 4. High Level Design

## Rol y Contexto
Eres un Arquitecto de Software Senior con experiencia diseñando sistemas
distribuidos y plataformas SaaS B2B de escala media. Has liderado sesiones
de arquitectura para equipos fullstack documentando decisiones técnicas con
el modelo C4, diagramas de secuencia y diagramas de flujo. Conoces los
patrones de arquitectura requeridos para implementar colaboración en tiempo
real (WebSockets, Server-Sent Events), motores de automatización basados
en triggers, integración con LLMs via API y pipelines de métricas para
dashboards de eficiencia.

## Tarea
Genera el diseño del sistema a alto nivel para las siguientes 4
funcionalidades nuevas del ATS, integrándolas coherentemente con la
arquitectura existente ya definida:

1. Eficiencia para departamentos de HR
2. Colaboración en tiempo real entre reclutadores y managers
3. Automatizaciones
4. Asistencia de IA en diversas tareas

## Contexto del sistema existente
- Arquitectura base: SaaS B2B multi-tenant, REST API
- Stack: Angular (frontend), Node.js/Express (backend),
  PostgreSQL (base de datos), AWS (infraestructura)
- Contenedores existentes: Angular SPA, Node.js API, PostgreSQL, Email Service
- Componentes backend existentes: AuthModule, JobOpeningModule,
  CandidateModule, PipelineModule, EvaluationModule, CommunicationModule
- Modelo de datos extendido: [PEGAR MODELO DE DATOS CON NUEVAS ENTIDADES]
- Casos de uso nuevos: [PEGAR CASOS DE USO DE LAS NUEVAS FUNCIONALIDADES]
- Diagramas C4 existentes: [PEGAR DIAGRAMAS C4 DEL DISEÑO BASE]

## Diagramas requeridos
Genera los siguientes diagramas en sintaxis Mermaid, en este orden:

### 1. Diagrama de contexto actualizado (C4Context)
- Mantener actores existentes: Recruiter, Candidate, Admin, HiringManager
- Agregar nuevos sistemas externos requeridos:
  LLM Provider (OpenAI / Anthropic), Motor de automatización,
  Servicio de notificaciones en tiempo real
- Mostrar relaciones nuevas entre actores y el sistema

### 2. Diagrama de contenedores actualizado (C4Container)
- Mantener contenedores existentes: Angular SPA, Node.js API,
  PostgreSQL, Email Service
- Agregar contenedores nuevos justificados por las funcionalidades:
  - WebSocket Server o SSE Service (colaboración en tiempo real)
  - Automation Engine (motor de reglas y triggers)
  - AI Service / LLM Gateway (capa de abstracción sobre proveedores IA)
  - Metrics & Analytics Service (eficiencia HR)
- Mostrar protocolos de comunicación entre todos los contenedores

### 3. Diagrama de componentes — módulos nuevos del backend (C4Component)
- Agregar dentro del boundary "Node.js API":
  - CollaborationModule
  - AutomationModule
  - AIAssistantModule
  - MetricsModule
- Mostrar dependencias entre módulos nuevos y existentes
- Mostrar relación de cada módulo nuevo con la base de datos
  y con los servicios externos correspondientes

### 4. Diagrama de secuencia — Automatización
Modela el flujo: Candidato avanza de etapa →
Sistema evalúa reglas de automatización →
Se dispara acción configurada (enviar email / mover etapa / notificar)

### 5. Diagrama de secuencia — Asistencia IA
Modela el flujo: Reclutador solicita sugerencia de IA sobre candidato →
AIAssistantModule construye contexto desde el modelo de datos →
Llamada al LLM Provider → Respuesta procesada →
Sugerencia almacenada y presentada al reclutador

### 6. Diagrama de secuencia — Colaboración en tiempo real
Modela el flujo: HiringManager comenta en una postulación →
Evento emitido via WebSocket →
Reclutador recibe notificación en tiempo real →
Actividad registrada en el modelo de datos

### 7. Diagrama de flujo — ciclo de vida de una Automatización
Estados: Configurada → Activa → Trigger detectado →
Condiciones evaluadas → Acción ejecutada → (Exitosa | Fallida) →
Historial registrado

## Instrucciones de formato
Para cada diagrama:
- Bloque de código Mermaid con sintaxis válida para mermaid.live
- Usar C4Context, C4Container y C4Component para los diagramas C4
- Después de cada diagrama: sección de decisiones arquitectónicas
  con el siguiente formato:

  **Decisiones clave**
  - Decisión tomada y alternativa descartada
  - Razón técnica o de negocio en una oración
  - Impacto en el modelo de datos o en contenedores existentes

- Usar H2 para cada diagrama
- Usar H3 para la sección de decisiones clave de cada diagrama
- El output debe ser Markdown válido y en español

## Restricciones
- No modificar los diagramas C4 existentes; solo extenderlos
- Mantener coherencia con los nombres de módulos y entidades
  ya definidos en el modelo de datos y casos de uso
- Máximo 12 nodos por diagrama para mantener legibilidad
- Los diagramas de secuencia deben usar actores con los mismos
  nombres que los roles definidos en el modelo de datos
- La capa AIAssistantModule debe modelarse como un gateway
  agnóstico al proveedor, nunca acoplado directamente a OpenAI
  o Anthropic en el diagrama
- No incluir decisiones de infraestructura AWS específicas
  (no ARNs, no nombres de servicios propietarios) en esta fase
- Los WebSockets o SSE deben modelarse como un contenedor
  separado, no embebido dentro del Node.js API existente

## Audiencia
El output será presentado al equipo técnico completo (frontend,
backend, DevOps) y al Product Owner en una sesión de arquitectura
para las nuevas funcionalidades. Debe ser comprensible para todos
los perfiles sin perder precisión técnica y debe poder usarse
directamente como input para el sprint de implementación.

## Criterio de éxito
Al terminar de leer el documento, cualquier developer del equipo
debe poder responder sin ambigüedad:

1. ¿Qué contenedor nuevo necesito levantar para soportar
   colaboración en tiempo real?
2. ¿Cómo se comunica el motor de automatización con el
   pipeline existente sin acoplarse a sus módulos internos?
3. ¿Dónde vive la lógica de IA y cómo se abstrae del proveedor?
4. ¿Qué flujo de datos recorre una sugerencia de IA desde
   que se solicita hasta que se almacena en el modelo de datos?