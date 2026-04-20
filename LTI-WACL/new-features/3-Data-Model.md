A continuación propongo una **extensión conservadora y retrocompatible** del modelo ATS, manteniendo intactos los contratos base y agregando solo campos y entidades estrictamente necesarios para soportar los nuevos UC de eficiencia HR, colaboración, automatización e IA definidos en tu archivo. La propuesta parte del modelo actual multi-tenant con `Company` como límite de tenant, `Application` como raíz transaccional y `PipelineStage`/`StageTransition` como fuente de verdad del pipeline, además de los nuevos UC 21–33 que piden ranking IA, reglas automáticas, comentarios, menciones, scorecards e historial de actividad. 

---

## Resumen de cambios

| Entidad                 | Tipo de cambio                 | Funcionalidad                                    | Impacto en modelo existente |
| ----------------------- | ------------------------------ | ------------------------------------------------ | --------------------------- |
| Candidate               | Extensión de entidad existente | Eficiencia HR / Asistencia IA                    | Bajo                        |
| Application             | Extensión de entidad existente | Eficiencia HR / Automatizaciones / Asistencia IA | Medio                       |
| PipelineStage           | Extensión de entidad existente | Automatizaciones                                 | Bajo                        |
| Evaluation              | Extensión de entidad existente | Colaboración / Evaluación                        | Medio                       |
| JobOpening              | Extensión de entidad existente | Eficiencia HR                                    | Bajo                        |
| SavedFilter             | Nueva entidad                  | Eficiencia HR                                    | Bajo                        |
| BulkOperation           | Nueva entidad                  | Eficiencia HR                                    | Medio                       |
| BulkOperationItem       | Nueva entidad                  | Eficiencia HR                                    | Bajo                        |
| ActivityEvent           | Nueva entidad                  | Colaboración en tiempo real / Eficiencia HR      | Medio                       |
| CandidateViewEvent      | Nueva entidad                  | Colaboración en tiempo real / Eficiencia HR      | Bajo                        |
| CommentThread           | Nueva entidad                  | Colaboración en tiempo real                      | Bajo                        |
| Comment                 | Nueva entidad                  | Colaboración en tiempo real                      | Medio                       |
| CommentMention          | Nueva entidad                  | Colaboración en tiempo real                      | Bajo                        |
| UserNotification        | Nueva entidad                  | Colaboración en tiempo real                      | Bajo                        |
| AutomationRule          | Nueva entidad                  | Automatizaciones                                 | Medio                       |
| AutomationCondition     | Nueva entidad                  | Automatizaciones                                 | Bajo                        |
| AutomationAction        | Nueva entidad                  | Automatizaciones                                 | Bajo                        |
| AutomationExecution     | Nueva entidad                  | Automatizaciones                                 | Alto                        |
| AutomationExecutionItem | Nueva entidad                  | Automatizaciones                                 | Medio                       |
| SlaPolicy               | Nueva entidad                  | Automatizaciones / Eficiencia HR                 | Medio                       |
| SlaBreachEvent          | Nueva entidad                  | Automatizaciones / Eficiencia HR                 | Bajo                        |
| AiSuggestion            | Nueva entidad                  | Asistencia IA                                    | Alto                        |
| AiGenerationRun         | Nueva entidad                  | Asistencia IA                                    | Medio                       |
| AiSuggestionFeedback    | Nueva entidad                  | Asistencia IA                                    | Bajo                        |
| ScorecardTemplate       | Nueva entidad                  | Colaboración en tiempo real                      | Medio                       |
| ScorecardCriterion      | Nueva entidad                  | Colaboración en tiempo real                      | Bajo                        |
| ScorecardResponse       | Nueva entidad                  | Colaboración en tiempo real                      | Medio                       |
| JsonRuleOperand         | Value Object nuevo             | Automatizaciones                                 | Bajo                        |
| LlmUsage                | Value Object nuevo             | Asistencia IA                                    | Bajo                        |

---

## Extensiones a entidades existentes

### Candidate

| Campo                   | Tipo                                 | Notas                                          |
| ----------------------- | ------------------------------------ | ---------------------------------------------- |
| ⚠️ `profileCompletion`  | integer                              | porcentaje 0–100 para resumen operativo        |
| ⚠️ `lastViewedAt`       | datetime?                            | última visualización por algún usuario interno |
| ⚠️ `aiSummary`          | text?                                | resumen vigente generado por IA                |
| ⚠️ `aiSummaryStatus`    | enum(Pending, Ready, Failed, Stale)? | estado del último resumen IA                   |
| ⚠️ `aiSummaryUpdatedAt` | datetime?                            | marca temporal del último resumen IA           |

**Motivo del cambio:** soportar UC-21 y UC-29 sin depender de joins complejos sobre logs o ejecuciones históricas.

**Compatibilidad:** retrocompatible.
**Migración:** sí, por adición de columnas nullable o con default.

---

### Application

| Campo                  | Tipo                                | Notas                                           |
| ---------------------- | ----------------------------------- | ----------------------------------------------- |
| ⚠️ `lastActivityAt`    | datetime?                           | última actividad relevante sobre la postulación |
| ⚠️ `lastViewedAt`      | datetime?                           | última visualización del detalle de postulación |
| ⚠️ `bulkUpdatedAt`     | datetime?                           | última operación masiva aplicada                |
| ⚠️ `bulkUpdateBatchId` | UUID?                               | referencia lógica a `BulkOperation`             |
| ⚠️ `stageEnteredAt`    | datetime?                           | inicio de permanencia en etapa actual para SLA  |
| ⚠️ `lastStageChangeAt` | datetime?                           | redundancia explícita para queries operativas   |
| ⚠️ `updatedByType`     | enum(User, Automation, AI, System)? | origen de la última actualización relevante     |
| ⚠️ `aiScore`           | decimal?                            | score vigente sugerido por IA                   |
| ⚠️ `aiRank`            | integer?                            | posición sugerida dentro de una vacante         |
| ⚠️ `aiExplanation`     | text?                               | justificación resumida del score                |
| ⚠️ `aiScoredAt`        | datetime?                           | fecha del score vigente                         |

**Motivo del cambio:** resolver lectura rápida para UC-22, UC-24, UC-25 y UC-26, manteniendo el historial detallado en entidades nuevas.

**Compatibilidad:** retrocompatible.
**Migración:** sí.

---

### PipelineStage

| Campo         | Tipo     | Notas                                   |
| ------------- | -------- | --------------------------------------- |
| ⚠️ `slaHours` | integer? | objetivo máximo de permanencia en etapa |

**Motivo del cambio:** permitir SLA simple por etapa sin obligar a una tabla adicional para escenarios básicos.

**Compatibilidad:** retrocompatible.
**Migración:** sí.

---

### Evaluation

| Campo                    | Tipo      | Notas                                               |
| ------------------------ | --------- | --------------------------------------------------- |
| ⚠️ `scorecardTemplateId` | UUID?     | plantilla aplicada, si corresponde                  |
| ⚠️ `submittedAt`         | datetime? | fecha de envío formal                               |
| ⚠️ `structuredScore`     | decimal?  | score agregado derivado de respuestas estructuradas |

**Motivo del cambio:** soportar UC-27 sin romper el contrato actual de evaluación.

**Compatibilidad:** retrocompatible.
**Migración:** sí.

---

### JobOpening

| Campo                           | Tipo     | Notas                                 |
| ------------------------------- | -------- | ------------------------------------- |
| ⚠️ `targetTimeToHireDays`       | integer? | métrica objetivo de eficiencia        |
| ⚠️ `defaultScorecardTemplateId` | UUID?    | plantilla por defecto para evaluación |

**Motivo del cambio:** mejorar configuración operacional por vacante y reducir joins de configuración.

**Compatibilidad:** retrocompatible.
**Migración:** sí.

---

## Nuevas entidades por módulo funcional

## Módulo: Eficiencia HR

### SavedFilter

**Tipo:** Nueva entidad

**Justificación:** permite persistir criterios de búsqueda reutilizables para recruiters y equipos, alineado con UC-23.

| Atributo      | Tipo                                     | Notas                  |
| ------------- | ---------------------------------------- | ---------------------- |
| `id`          | UUID                                     | PK                     |
| `companyId`   | UUID                                     | FK → Company           |
| `ownerUserId` | UUID                                     | FK → User              |
| `module`      | enum(Candidate, Application, JobOpening) | contexto funcional     |
| `name`        | string                                   | único por owner+module |
| `visibility`  | enum(Private, Shared)                    | alcance                |
| `filterJson`  | jsonb                                    | criterios serializados |
| `sortJson`    | jsonb?                                   | orden persistido       |
| `createdAt`   | datetime                                 |                        |
| `updatedAt`   | datetime                                 |                        |

**Relaciones**

* `SavedFilter N : 1 Company`
* `SavedFilter N : 1 User`

**Reglas de negocio**

* El `name` debe ser único por `ownerUserId + module`.
* Un filtro `Shared` solo puede ser creado por roles autorizados.
* `filterJson` no puede estar vacío.
* El filtro nunca puede referenciar datos de otra `Company`.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Eficiencia para departamentos de HR

---

### BulkOperation

**Tipo:** Nueva entidad

**Justificación:** agrupa una acción masiva como unidad auditable y consultable para UC-22.

| Atributo            | Tipo                                                                     | Notas            |
| ------------------- | ------------------------------------------------------------------------ | ---------------- |
| `id`                | UUID                                                                     | PK               |
| `companyId`         | UUID                                                                     | FK → Company     |
| `requestedByUserId` | UUID                                                                     | FK → User        |
| `targetType`        | enum(Application, Candidate)                                             | alcance del lote |
| `operationType`     | enum(MoveStage, Reject, Tag, Assign, Archive)                            | acción ejecutada |
| `payloadJson`       | jsonb                                                                    | parámetros       |
| `status`            | enum(Pending, Running, Completed, PartiallyCompleted, Failed, Cancelled) |                  |
| `requestedAt`       | datetime                                                                 |                  |
| `completedAt`       | datetime?                                                                |                  |
| `successCount`      | integer                                                                  | default 0        |
| `failureCount`      | integer                                                                  | default 0        |

**Relaciones**

* `BulkOperation N : 1 Company`
* `BulkOperation N : 1 User`
* `BulkOperation 1 : N BulkOperationItem`

**Reglas de negocio**

* El lote no puede mezclar entidades de distintas compañías.
* `payloadJson` debe ser coherente con `operationType`.
* Un lote `Completed` no puede reabrirse.
* Debe conservarse aunque algunas filas fallen.

**Clasificación DDD:** Aggregate Root

**Funcionalidad que habilita:** Eficiencia para departamentos de HR

---

### BulkOperationItem

**Tipo:** Nueva entidad

**Justificación:** permite trazabilidad por registro afectado dentro de una operación masiva.

| Atributo          | Tipo                                    | Notas                         |
| ----------------- | --------------------------------------- | ----------------------------- |
| `id`              | UUID                                    | PK                            |
| `bulkOperationId` | UUID                                    | FK → BulkOperation            |
| `targetId`        | UUID                                    | id de Application o Candidate |
| `status`          | enum(Pending, Success, Failed, Skipped) |                               |
| `message`         | string?                                 | detalle de resultado          |
| `processedAt`     | datetime?                               |                               |

**Relaciones**

* `BulkOperationItem N : 1 BulkOperation`

**Reglas de negocio**

* Cada item pertenece a un solo lote.
* `targetId` debe corresponder con `BulkOperation.targetType`.
* Solo un resultado final por item.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Eficiencia para departamentos de HR

---

## Módulo: Colaboración en tiempo real

### ActivityEvent

**Tipo:** Nueva entidad

**Justificación:** centraliza actividad cronológica para responder quién vio, comentó, movió o automatizó una postulación sin inferencias complejas. Esto se alinea con la necesidad de trazabilidad y visibilidad del historial descrita para UC-21, UC-28, UC-31 y UC-32. 

| Atributo          | Tipo                                                                                                                | Notas                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `id`              | UUID                                                                                                                | PK                           |
| `companyId`       | UUID                                                                                                                | FK → Company                 |
| `contextType`     | enum(Application, Candidate, JobOpening)                                                                            |                              |
| `contextId`       | UUID                                                                                                                |                              |
| `eventType`       | enum(Viewed, Commented, Mentioned, StageChanged, EvaluationSubmitted, BulkUpdated, AutomationExecuted, AiSuggested) |                              |
| `actorType`       | enum(User, Automation, AI, System)                                                                                  |                              |
| `actorUserId`     | UUID?                                                                                                               | FK → User, null si no aplica |
| `metadataJson`    | jsonb?                                                                                                              | payload corto                |
| `visibilityScope` | enum(Internal, HiringTeam, AdminOnly)                                                                               |                              |
| `occurredAt`      | datetime                                                                                                            |                              |

**Relaciones**

* `ActivityEvent N : 1 Company`
* `ActivityEvent N : 1 User` (opcional)

**Reglas de negocio**

* `contextType + contextId` siempre deben apuntar a una entidad del mismo tenant.
* `actorUserId` es obligatorio cuando `actorType = User`.
* `occurredAt` es inmutable.
* `metadataJson` no debe contener contenido fuente sensible redundante.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### CandidateViewEvent

**Tipo:** Nueva entidad

**Justificación:** desacopla la analítica de visualización del timeline general y permite métricas precisas de “quién ha visto y cuándo”.

| Atributo       | Tipo                                                           | Notas        |
| -------------- | -------------------------------------------------------------- | ------------ |
| `id`           | UUID                                                           | PK           |
| `companyId`    | UUID                                                           | FK → Company |
| `viewerUserId` | UUID                                                           | FK → User    |
| `contextType`  | enum(Candidate, Application)                                   |              |
| `contextId`    | UUID                                                           |              |
| `viewedAt`     | datetime                                                       |              |
| `sourceModule` | enum(CandidateList, CandidateDetail, PipelineBoard, Dashboard) |              |

**Relaciones**

* `CandidateViewEvent N : 1 Company`
* `CandidateViewEvent N : 1 User`

**Reglas de negocio**

* Solo usuarios activos pueden generar vistas.
* No consolida ni sobreescribe eventos previos.
* Toda vista debe respetar tenant.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### CommentThread

**Tipo:** Nueva entidad

**Justificación:** separa el hilo colaborativo del contenido del comentario y permite extender comentarios en tiempo real sin sobrecargar `Note`.

| Atributo           | Tipo                         | Notas        |
| ------------------ | ---------------------------- | ------------ |
| `id`               | UUID                         | PK           |
| `companyId`        | UUID                         | FK → Company |
| `contextType`      | enum(Application, Candidate) |              |
| `contextId`        | UUID                         |              |
| `status`           | enum(Open, Resolved)         |              |
| `createdByUserId`  | UUID                         | FK → User    |
| `createdAt`        | datetime                     |              |
| `resolvedAt`       | datetime?                    |              |
| `resolvedByUserId` | UUID?                        | FK → User    |

**Relaciones**

* `CommentThread N : 1 Company`
* `CommentThread N : 1 User` (creator)
* `CommentThread 1 : N Comment`

**Reglas de negocio**

* Un thread no puede cruzar contextos.
* Solo usuarios autorizados pueden resolver un thread.
* Un thread `Resolved` puede reabrirse solo con trazabilidad.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### Comment

**Tipo:** Nueva entidad

**Justificación:** soporta comentarios colaborativos distintos de `Note`, con visibilidad, menciones y trazabilidad.

| Atributo          | Tipo                                  | Notas              |
| ----------------- | ------------------------------------- | ------------------ |
| `id`              | UUID                                  | PK                 |
| `companyId`       | UUID                                  | FK → Company       |
| `threadId`        | UUID                                  | FK → CommentThread |
| `authorUserId`    | UUID                                  | FK → User          |
| `body`            | text                                  |                    |
| `visibilityScope` | enum(Internal, HiringTeam, AdminOnly) |                    |
| `createdAt`       | datetime                              |                    |
| `editedAt`        | datetime?                             |                    |
| `deletedAt`       | datetime?                             | soft delete        |

**Relaciones**

* `Comment N : 1 Company`
* `Comment N : 1 CommentThread`
* `Comment N : 1 User`
* `Comment 1 : N CommentMention`

**Reglas de negocio**

* `body` no puede estar vacío.
* El autor no cambia.
* `deletedAt` no elimina menciones históricas.
* Todo comentario hereda el contexto del thread.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### CommentMention

**Tipo:** Nueva entidad

**Justificación:** permite direccionar colaboración puntual y rastrear a quién se mencionó en cada comentario, según UC-32. El archivo establece que las menciones deben quedar trazadas individualmente y solo pueden dirigirse a usuarios activos del mismo tenant. 

| Atributo             | Tipo                           | Notas        |
| -------------------- | ------------------------------ | ------------ |
| `id`                 | UUID                           | PK           |
| `commentId`          | UUID                           | FK → Comment |
| `mentionedUserId`    | UUID                           | FK → User    |
| `createdAt`          | datetime                       |              |
| `notificationStatus` | enum(Pending, Delivered, Read) |              |

**Relaciones**

* `CommentMention N : 1 Comment`
* `CommentMention N : 1 User`

**Reglas de negocio**

* Solo se puede mencionar un `User` activo de la misma `Company`.
* Una misma combinación `commentId + mentionedUserId` no se duplica.
* La mención no asigna ownership por sí sola.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### UserNotification

**Tipo:** Nueva entidad

**Justificación:** soporta entrega interna de menciones, cambios y alertas sin integrar canales externos fuera del alcance MVP.

| Atributo     | Tipo                                                                                           | Notas        |
| ------------ | ---------------------------------------------------------------------------------------------- | ------------ |
| `id`         | UUID                                                                                           | PK           |
| `companyId`  | UUID                                                                                           | FK → Company |
| `userId`     | UUID                                                                                           | FK → User    |
| `type`       | enum(Mention, CommentReply, AutomationAlert, SlaAlert, EvaluationSubmitted, AiSuggestionReady) |              |
| `sourceType` | enum(CommentMention, Comment, AutomationExecution, SlaBreachEvent, AiSuggestion)               |              |
| `sourceId`   | UUID                                                                                           |              |
| `status`     | enum(Unread, Read, Dismissed)                                                                  |              |
| `createdAt`  | datetime                                                                                       |              |
| `readAt`     | datetime?                                                                                      |              |

**Relaciones**

* `UserNotification N : 1 Company`
* `UserNotification N : 1 User`

**Reglas de negocio**

* Toda notificación pertenece a un único usuario.
* `readAt` solo existe cuando `status = Read`.
* No se crean notificaciones cross-tenant.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### ScorecardTemplate

**Tipo:** Nueva entidad

**Justificación:** formaliza scorecards reutilizables por vacante para UC-27.

| Atributo          | Tipo                   | Notas        |
| ----------------- | ---------------------- | ------------ |
| `id`              | UUID                   | PK           |
| `companyId`       | UUID                   | FK → Company |
| `name`            | string                 |              |
| `status`          | enum(Active, Inactive) |              |
| `createdByUserId` | UUID                   | FK → User    |
| `createdAt`       | datetime               |              |

**Relaciones**

* `ScorecardTemplate N : 1 Company`
* `ScorecardTemplate N : 1 User`
* `ScorecardTemplate 1 : N ScorecardCriterion`
* `ScorecardTemplate 1 : N Evaluation`

**Reglas de negocio**

* Debe tener al menos un criterio activo.
* Un template inactivo no puede asignarse a nuevas vacantes.
* El template pertenece a una sola compañía.

**Clasificación DDD:** Aggregate Root

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### ScorecardCriterion

**Tipo:** Nueva entidad

**Justificación:** representa criterios estructurados y ordenados del scorecard.

| Atributo              | Tipo     | Notas                  |
| --------------------- | -------- | ---------------------- |
| `id`                  | UUID     | PK                     |
| `scorecardTemplateId` | UUID     | FK → ScorecardTemplate |
| `name`                | string   |                        |
| `description`         | string?  |                        |
| `sequence`            | integer  |                        |
| `isRequired`          | boolean  |                        |
| `weight`              | decimal? | opcional               |
| `scaleMin`            | integer? |                        |
| `scaleMax`            | integer? |                        |

**Relaciones**

* `ScorecardCriterion N : 1 ScorecardTemplate`
* `ScorecardCriterion 1 : N ScorecardResponse`

**Reglas de negocio**

* `sequence` único por template.
* `weight` debe ser positivo cuando exista.
* `scaleMin <= scaleMax`.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

### ScorecardResponse

**Tipo:** Nueva entidad

**Justificación:** permite almacenar respuestas estructuradas por evaluación y criterio.

| Atributo       | Tipo     | Notas                   |
| -------------- | -------- | ----------------------- |
| `id`           | UUID     | PK                      |
| `evaluationId` | UUID     | FK → Evaluation         |
| `criterionId`  | UUID     | FK → ScorecardCriterion |
| `score`        | decimal? |                         |
| `comment`      | text?    |                         |
| `createdAt`    | datetime |                         |

**Relaciones**

* `ScorecardResponse N : 1 Evaluation`
* `ScorecardResponse N : 1 ScorecardCriterion`

**Reglas de negocio**

* Solo una respuesta por `evaluationId + criterionId`.
* Los criterios requeridos deben existir antes de `Evaluation.submittedAt`.
* `score` debe respetar la escala del criterio.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Colaboración en tiempo real entre reclutadores y managers

---

## Módulo: Automatizaciones

### AutomationRule

**Tipo:** Nueva entidad

**Justificación:** modela reglas configurables por tenant para UC-30 y sirve como raíz estable del agregado de automatización. El caso de uso exige un disparador, condiciones y acción, y la regla puede quedar activa o inactiva sin hardcodear lógica en el esquema. 

| Atributo          | Tipo                                                                                                                                      | Notas                     |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| `id`              | UUID                                                                                                                                      | PK                        |
| `companyId`       | UUID                                                                                                                                      | FK → Company              |
| `name`            | string                                                                                                                                    |                           |
| `description`     | string?                                                                                                                                   |                           |
| `scopeType`       | enum(Global, JobOpening, PipelineStage)                                                                                                   |                           |
| `scopeId`         | UUID?                                                                                                                                     | id contextual según scope |
| `triggerType`     | enum(ApplicationCreated, StageChanged, EvaluationSubmitted, SlaBreached, AiSuggestionAccepted, CommunicationFailed, ScheduledTimeReached) |                           |
| `status`          | enum(Active, Inactive)                                                                                                                    |                           |
| `createdByUserId` | UUID                                                                                                                                      | FK → User                 |
| `createdAt`       | datetime                                                                                                                                  |                           |
| `updatedAt`       | datetime                                                                                                                                  |                           |

**Relaciones**

* `AutomationRule N : 1 Company`
* `AutomationRule N : 1 User`
* `AutomationRule 1 : N AutomationCondition`
* `AutomationRule 1 : N AutomationAction`
* `AutomationRule 1 : N AutomationExecution`

**Reglas de negocio**

* Debe existir al menos una acción.
* La regla inactiva no ejecuta.
* `scopeId` es obligatorio cuando `scopeType != Global`.
* Toda regla aplica solo dentro de su `Company`.

**Clasificación DDD:** Aggregate Root

**Funcionalidad que habilita:** Automatizaciones

---

### AutomationCondition

**Tipo:** Nueva entidad

**Justificación:** desacopla condiciones evaluables de la regla principal.

| Atributo           | Tipo                                                                    | Notas                  |
| ------------------ | ----------------------------------------------------------------------- | ---------------------- |
| `id`               | UUID                                                                    | PK                     |
| `automationRuleId` | UUID                                                                    | FK → AutomationRule    |
| `sequence`         | integer                                                                 | orden de evaluación    |
| `fieldPath`        | string                                                                  | ej. application.status |
| `operator`         | enum(Eq, Neq, Gt, Gte, Lt, Lte, In, NotIn, Exists, ChangedTo, Contains) |                        |
| `valueJson`        | jsonb                                                                   | operando               |
| `logicalJoin`      | enum(AND, OR)                                                           | unión con la siguiente |

**Relaciones**

* `AutomationCondition N : 1 AutomationRule`

**Reglas de negocio**

* `sequence` único por regla.
* `valueJson` debe ser compatible con `operator`.
* `logicalJoin` no aplica en la última condición pero puede persistirse por uniformidad.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Automatizaciones

---

### AutomationAction

**Tipo:** Nueva entidad

**Justificación:** define acciones ejecutables configurables sin hardcodear reglas por tenant.

| Atributo           | Tipo                                                                                                             | Notas               |
| ------------------ | ---------------------------------------------------------------------------------------------------------------- | ------------------- |
| `id`               | UUID                                                                                                             | PK                  |
| `automationRuleId` | UUID                                                                                                             | FK → AutomationRule |
| `sequence`         | integer                                                                                                          |                     |
| `actionType`       | enum(UpdateApplicationStatus, MoveToStage, CreateNotification, CreateTask, ScheduleCommunication, MarkSlaBreach) |                     |
| `configJson`       | jsonb                                                                                                            | parámetros          |

**Relaciones**

* `AutomationAction N : 1 AutomationRule`

**Reglas de negocio**

* `sequence` único por regla.
* `configJson` debe cumplir el contrato de `actionType`.
* Las acciones no pueden cruzar tenant.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Automatizaciones

---

### AutomationExecution

**Tipo:** Nueva entidad

**Justificación:** responde directamente qué automatizaciones se dispararon, sobre qué contexto y con qué resultado, que es uno de los criterios de éxito del modelo. El archivo pide poder consultar disparos de automatización por vacante y resultado sin lógica fuera del esquema. 

| Atributo            | Tipo                                                                   | Notas                |
| ------------------- | ---------------------------------------------------------------------- | -------------------- |
| `id`                | UUID                                                                   | PK                   |
| `companyId`         | UUID                                                                   | FK → Company         |
| `automationRuleId`  | UUID                                                                   | FK → AutomationRule  |
| `jobOpeningId`      | UUID?                                                                  | FK → JobOpening      |
| `applicationId`     | UUID?                                                                  | FK → Application     |
| `triggeredByType`   | enum(System, User, Scheduler)                                          |                      |
| `triggeredByUserId` | UUID?                                                                  | FK → User            |
| `triggerEventType`  | string                                                                 | evento materializado |
| `status`            | enum(Skipped, Running, Succeeded, PartiallySucceeded, Failed, Blocked) |                      |
| `resultSummary`     | string?                                                                |                      |
| `startedAt`         | datetime                                                               |                      |
| `finishedAt`        | datetime?                                                              |                      |

**Relaciones**

* `AutomationExecution N : 1 Company`
* `AutomationExecution N : 1 AutomationRule`
* `AutomationExecution N : 1 JobOpening` (opcional)
* `AutomationExecution N : 1 Application` (opcional)
* `AutomationExecution 1 : N AutomationExecutionItem`

**Reglas de negocio**

* Debe existir al menos `jobOpeningId` o `applicationId`.
* `finishedAt` obligatorio en estados terminales.
* El `status` es inmutable una vez terminal, salvo compensación explícita con nueva ejecución.

**Clasificación DDD:** Aggregate Root

**Funcionalidad que habilita:** Automatizaciones

---

### AutomationExecutionItem

**Tipo:** Nueva entidad

**Justificación:** descompone el resultado por acción ejecutada dentro de una ejecución.

| Atributo                | Tipo                                      | Notas                    |
| ----------------------- | ----------------------------------------- | ------------------------ |
| `id`                    | UUID                                      | PK                       |
| `automationExecutionId` | UUID                                      | FK → AutomationExecution |
| `automationActionId`    | UUID                                      | FK → AutomationAction    |
| `status`                | enum(Succeeded, Failed, Skipped, Blocked) |                          |
| `message`               | string?                                   |                          |
| `payloadJson`           | jsonb?                                    | resultado puntual        |
| `executedAt`            | datetime                                  |                          |

**Relaciones**

* `AutomationExecutionItem N : 1 AutomationExecution`
* `AutomationExecutionItem N : 1 AutomationAction`

**Reglas de negocio**

* Una fila por acción ejecutada por ejecución.
* `payloadJson` no debe duplicar todo el estado del agregado.
* `executedAt` es obligatorio.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Automatizaciones

---

### SlaPolicy

**Tipo:** Nueva entidad

**Justificación:** complementa `PipelineStage.slaHours` para políticas avanzadas por vacante, departamento o etapa sin romper el modelo base.

| Atributo          | Tipo                   | Notas              |
| ----------------- | ---------------------- | ------------------ |
| `id`              | UUID                   | PK                 |
| `companyId`       | UUID                   | FK → Company       |
| `jobOpeningId`    | UUID?                  | FK → JobOpening    |
| `pipelineStageId` | UUID?                  | FK → PipelineStage |
| `name`            | string                 |                    |
| `thresholdHours`  | integer                |                    |
| `warningHours`    | integer?               |                    |
| `status`          | enum(Active, Inactive) |                    |
| `createdAt`       | datetime               |                    |

**Relaciones**

* `SlaPolicy N : 1 Company`
* `SlaPolicy N : 1 JobOpening` (opcional)
* `SlaPolicy N : 1 PipelineStage` (opcional)
* `SlaPolicy 1 : N SlaBreachEvent`

**Reglas de negocio**

* Debe existir `jobOpeningId` o `pipelineStageId`.
* `warningHours` debe ser menor que `thresholdHours` cuando exista.
* Solo políticas activas participan en monitoreo.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Automatizaciones

---

### SlaBreachEvent

**Tipo:** Nueva entidad

**Justificación:** registra incumplimientos concretos para alertas y métricas operativas.

| Atributo          | Tipo                               | Notas              |
| ----------------- | ---------------------------------- | ------------------ |
| `id`              | UUID                               | PK                 |
| `companyId`       | UUID                               | FK → Company       |
| `slaPolicyId`     | UUID                               | FK → SlaPolicy     |
| `applicationId`   | UUID                               | FK → Application   |
| `jobOpeningId`    | UUID                               | FK → JobOpening    |
| `pipelineStageId` | UUID                               | FK → PipelineStage |
| `status`          | enum(Open, Acknowledged, Resolved) |                    |
| `breachedAt`      | datetime                           |                    |
| `resolvedAt`      | datetime?                          |                    |

**Relaciones**

* `SlaBreachEvent N : 1 Company`
* `SlaBreachEvent N : 1 SlaPolicy`
* `SlaBreachEvent N : 1 Application`
* `SlaBreachEvent N : 1 JobOpening`
* `SlaBreachEvent N : 1 PipelineStage`

**Reglas de negocio**

* Una brecha abierta por aplicación+etapa no debe duplicarse.
* `resolvedAt` requiere `status = Resolved`.
* Toda brecha referencia una policy válida.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Automatizaciones

---

## Módulo: Asistencia IA

### AiGenerationRun

**Tipo:** Nueva entidad

**Justificación:** registra cada ejecución con LLM de manera auditable y neutral al proveedor, tal como exigen las restricciones del archivo. Debe guardar `provider`, `modelVersion` y métricas de tokens sin asumir proveedor concreto. 

| Atributo           | Tipo                                                                                 | Notas                      |
| ------------------ | ------------------------------------------------------------------------------------ | -------------------------- |
| `id`               | UUID                                                                                 | PK                         |
| `companyId`        | UUID                                                                                 | FK → Company               |
| `contextType`      | enum(Candidate, Application, Interview, JobOpening)                                  |                            |
| `contextId`        | UUID                                                                                 |                            |
| `purpose`          | enum(ResumeSummary, CandidateRanking, InterviewQuestions, RecommendationExplanation) |                            |
| `provider`         | string                                                                               | genérico                   |
| `modelVersion`     | string                                                                               | genérico                   |
| `promptText`       | text                                                                                 | prompt materializado       |
| `responseText`     | text?                                                                                | respuesta cruda o resumida |
| `promptTokens`     | integer?                                                                             |                            |
| `completionTokens` | integer?                                                                             |                            |
| `status`           | enum(Pending, Succeeded, Failed, Cancelled)                                          |                            |
| `startedAt`        | datetime                                                                             |                            |
| `completedAt`      | datetime?                                                                            |                            |

**Relaciones**

* `AiGenerationRun N : 1 Company`
* `AiGenerationRun 1 : N AiSuggestion`

**Reglas de negocio**

* Toda ejecución debe tener `purpose`.
* `provider` y `modelVersion` son obligatorios en ejecuciones iniciadas.
* Las ejecuciones fallidas pueden no tener `responseText`.
* No se usa para persistir secretos ni credenciales.

**Clasificación DDD:** Aggregate Root

**Funcionalidad que habilita:** Asistencia de IA en diversas tareas

---

### AiSuggestion

**Tipo:** Nueva entidad

**Justificación:** representa una sugerencia consumible por negocio, separada del log de ejecución, para responder qué sugirió la IA y si fue aceptado o no. Esto cubre ranking, resumen y preguntas sugeridas de los UC de IA. 

| Atributo            | Tipo                                                                                     | Notas                 |
| ------------------- | ---------------------------------------------------------------------------------------- | --------------------- |
| `id`                | UUID                                                                                     | PK                    |
| `companyId`         | UUID                                                                                     | FK → Company          |
| `aiGenerationRunId` | UUID                                                                                     | FK → AiGenerationRun  |
| `contextType`       | enum(Candidate, Application, Interview)                                                  |                       |
| `contextId`         | UUID                                                                                     |                       |
| `suggestionType`    | enum(ResumeSummary, CandidateScore, CandidateRank, InterviewQuestionSet, Recommendation) |                       |
| `title`             | string?                                                                                  |                       |
| `contentJson`       | jsonb                                                                                    | payload de sugerencia |
| `status`            | enum(Generated, Accepted, Rejected, Superseded, Expired)                                 |                       |
| `acceptedByUserId`  | UUID?                                                                                    | FK → User             |
| `acceptedAt`        | datetime?                                                                                |                       |
| `createdAt`         | datetime                                                                                 |                       |

**Relaciones**

* `AiSuggestion N : 1 Company`
* `AiSuggestion N : 1 AiGenerationRun`
* `AiSuggestion N : 1 User` (acceptedBy, opcional)
* `AiSuggestion 1 : N AiSuggestionFeedback`

**Reglas de negocio**

* Una sugerencia aceptada debe tener `acceptedByUserId` y `acceptedAt`.
* `contentJson` debe cumplir el contrato de `suggestionType`.
* La aceptación no siempre implica escritura automática en la entidad de dominio; puede requerir aplicación explícita.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Asistencia de IA en diversas tareas

---

### AiSuggestionFeedback

**Tipo:** Nueva entidad

**Justificación:** captura aceptación, rechazo y feedback explícito para mejorar trazabilidad y UX.

| Atributo         | Tipo                                         | Notas             |
| ---------------- | -------------------------------------------- | ----------------- |
| `id`             | UUID                                         | PK                |
| `aiSuggestionId` | UUID                                         | FK → AiSuggestion |
| `userId`         | UUID                                         | FK → User         |
| `decision`       | enum(Accepted, Rejected, EditedBeforeAccept) |                   |
| `feedbackText`   | text?                                        |                   |
| `createdAt`      | datetime                                     |                   |

**Relaciones**

* `AiSuggestionFeedback N : 1 AiSuggestion`
* `AiSuggestionFeedback N : 1 User`

**Reglas de negocio**

* Debe existir una decisión por fila.
* Puede haber múltiples feedbacks, pero solo una aceptación final vigente.
* El usuario debe pertenecer a la misma compañía.

**Clasificación DDD:** Entity

**Funcionalidad que habilita:** Asistencia de IA en diversas tareas

---

## Value Objects nuevos

### JsonRuleOperand

**Tipo:** Value Object nuevo

**Justificación:** encapsula estructura de operandos configurables reutilizada en `AutomationCondition.valueJson` y `AutomationAction.configJson`.

| Atributo    | Tipo   | Notas                               |
| ----------- | ------ | ----------------------------------- |
| `kind`      | string | scalar, array, range, reference     |
| `value`     | jsonb  | contenido                           |
| `valueType` | string | string, number, boolean, date, uuid |

**Relaciones**

* Embebido en `AutomationCondition`
* Embebido en `AutomationAction`

**Reglas de negocio**

* `valueType` debe ser coherente con `value`.
* `kind = range` requiere dos extremos.

**Clasificación DDD:** Value Object

**Funcionalidad que habilita:** Automatizaciones

---

### LlmUsage

**Tipo:** Value Object nuevo

**Justificación:** encapsula telemetría reutilizable de uso del modelo en ejecuciones IA.

| Atributo           | Tipo    | Notas |
| ------------------ | ------- | ----- |
| `provider`         | string  |       |
| `modelVersion`     | string  |       |
| `promptTokens`     | integer |       |
| `completionTokens` | integer |       |

**Relaciones**

* Embebido conceptualmente en `AiGenerationRun`

**Reglas de negocio**

* Tokens no negativos.
* `provider` y `modelVersion` obligatorios.

**Clasificación DDD:** Value Object

**Funcionalidad que habilita:** Asistencia de IA en diversas tareas

---

## Aggregate Roots nuevos o promovidos

| Entidad             | Motivo                                                                                   |
| ------------------- | ---------------------------------------------------------------------------------------- |
| BulkOperation       | necesita encapsular items, estado global y auditoría del lote como una unidad de negocio |
| AutomationRule      | es la unidad de configuración administrable por tenant, con condiciones y acciones hijas |
| AutomationExecution | es la unidad auditable de disparo y resultado, clave para reporting y troubleshooting    |
| AiGenerationRun     | encapsula ciclo completo de invocación IA y desacopla logs de sugerencias aplicables     |
| ScorecardTemplate   | gobierna criterios reutilizables y su evolución por compañía                             |

---

## Decisiones de modelado clave

### Eficiencia HR

**Alternativas consideradas**

* Guardar todo en campos calculados dentro de `Application`
* Agregar entidades específicas para filtros y lotes

**Decisión tomada**
Agregar `SavedFilter`, `BulkOperation` y `BulkOperationItem`, y solo unos pocos campos denormalizados en `Application`.

**Razón técnica o de negocio**
Permite trazabilidad y reintentos sin contaminar el agregado `Application` con semántica de batch.

---

### Colaboración en tiempo real

**Alternativas consideradas**

* Reutilizar `Note` para comentarios colaborativos
* Crear un subdominio de colaboración separado

**Decisión tomada**
Mantener `Note` como comentario interno simple y crear `CommentThread`, `Comment`, `CommentMention`, `UserNotification` y `ActivityEvent`.

**Razón técnica o de negocio**
`Note` ya tiene un contrato claro y minimalista; colaboración en tiempo real necesita visibilidad, menciones, lectura y timeline, que son preocupaciones diferentes.

---

### Automatizaciones

**Alternativas consideradas**

* Hardcodear reglas en servicios
* Persistir una única tabla `AutomationRule` con JSON monolítico

**Decisión tomada**
Usar agregado `AutomationRule` + `AutomationCondition` + `AutomationAction`, y separado `AutomationExecution` + `AutomationExecutionItem`.

**Razón técnica o de negocio**
Separa configuración de ejecución, mejora auditabilidad y permite queries directas sobre disparos y resultados por vacante.

---

### Asistencia IA

**Alternativas consideradas**

* Guardar solo columnas en `Candidate`, `Application` e `Interview`
* Guardar solo un log genérico de prompts

**Decisión tomada**
Combinar columnas denormalizadas de lectura rápida con `AiGenerationRun`, `AiSuggestion` y `AiSuggestionFeedback`.

**Razón técnica o de negocio**
Se resuelve UX rápida y, al mismo tiempo, se mantiene historial neutral al proveedor, auditable y apto para aceptación/rechazo humano.

---
## EDR - Nuevas Integraciones

erDiagram

    %% =========================
    %% Tenant and Administration
    %% =========================
    Company {
        uuid id PK
        string name
        string slug
        string status
        datetime createdAt
    }

    User {
        uuid id PK
        uuid companyId FK
        string fullName
        string email
        string role
        string status
        datetime createdAt
    }

    %% =========================
    %% Candidate Management
    %% =========================
    Candidate {
        uuid id PK
        uuid companyId FK
        string firstName
        string lastName
        string email
        string phone
        string currentLocation
        string resumeUrl
        int profileCompletion
        datetime lastViewedAt
        text aiSummary
        string aiSummaryStatus
        datetime aiSummaryUpdatedAt
        datetime createdAt
    }

    SavedFilter {
        uuid id PK
        uuid companyId FK
        uuid ownerUserId FK
        string module
        string name
        string visibility
        text filterJson
        text sortJson
        datetime createdAt
        datetime updatedAt
    }

    CandidateViewEvent {
        uuid id PK
        uuid companyId FK
        uuid viewerUserId FK
        string contextType
        uuid contextId FK
        string sourceModule
        datetime viewedAt
    }

    %% =========================
    %% Job Openings and Pipeline
    %% =========================
    JobOpening {
        uuid id PK
        uuid companyId FK
        string title
        string department
        string location
        string workMode
        text description
        string status
        uuid sourceTemplateId FK
        uuid createdByUserId FK
        int targetTimeToHireDays
        uuid defaultScorecardTemplateId FK
        datetime createdAt
        datetime closedAt
    }

    PipelineTemplate {
        uuid id PK
        uuid companyId FK
        string name
        boolean isDefault
        datetime createdAt
    }

    PipelineStage {
        uuid id PK
        string ownerType
        uuid ownerId FK
        string name
        int sequence
        string stageType
        int slaHours
    }

    Application {
        uuid id PK
        uuid companyId FK
        uuid candidateId FK
        uuid jobOpeningId FK
        uuid currentStageId FK
        datetime stageUpdatedAt
        string status
        datetime appliedAt
        string source
        string rejectionReason
        datetime lastActivityAt
        datetime lastViewedAt
        datetime bulkUpdatedAt
        uuid bulkUpdateBatchId FK
        datetime stageEnteredAt
        datetime lastStageChangeAt
        string updatedByType
        decimal aiScore
        int aiRank
        text aiExplanation
        datetime aiScoredAt
    }

    StageTransition {
        uuid id PK
        uuid applicationId FK
        uuid fromStageId FK
        uuid toStageId FK
        uuid changedByUserId FK
        datetime changedAt
        string comment
    }

    BulkOperation {
        uuid id PK
        uuid companyId FK
        uuid requestedByUserId FK
        string targetType
        string operationType
        text payloadJson
        string status
        int successCount
        int failureCount
        datetime requestedAt
        datetime completedAt
    }

    BulkOperationItem {
        uuid id PK
        uuid bulkOperationId FK
        uuid targetId FK
        string status
        string message
        datetime processedAt
    }

    SlaPolicy {
        uuid id PK
        uuid companyId FK
        uuid jobOpeningId FK
        uuid pipelineStageId FK
        string name
        int thresholdHours
        int warningHours
        string status
        datetime createdAt
    }

    SlaBreachEvent {
        uuid id PK
        uuid companyId FK
        uuid slaPolicyId FK
        uuid applicationId FK
        uuid jobOpeningId FK
        uuid pipelineStageId FK
        string status
        datetime breachedAt
        datetime resolvedAt
    }

    %% =========================
    %% Evaluation and Collaboration
    %% =========================
    Evaluation {
        uuid id PK
        uuid applicationId FK
        uuid evaluatorUserId FK
        uuid stageId FK
        uuid scorecardTemplateId FK
        decimal score
        decimal structuredScore
        string recommendation
        text summary
        datetime createdAt
        datetime completedAt
        datetime submittedAt
    }

    ScorecardTemplate {
        uuid id PK
        uuid companyId FK
        string name
        string status
        uuid createdByUserId FK
        datetime createdAt
    }

    ScorecardCriterion {
        uuid id PK
        uuid scorecardTemplateId FK
        string name
        string description
        int sequence
        boolean isRequired
        decimal weight
        int scaleMin
        int scaleMax
    }

    ScorecardResponse {
        uuid id PK
        uuid evaluationId FK
        uuid criterionId FK
        decimal score
        text comment
        datetime createdAt
    }

    Note {
        uuid id PK
        string contextType
        uuid contextId FK
        uuid authorUserId FK
        text content
        datetime createdAt
    }

    CommentThread {
        uuid id PK
        uuid companyId FK
        string contextType
        uuid contextId FK
        string status
        uuid createdByUserId FK
        uuid resolvedByUserId FK
        datetime createdAt
        datetime resolvedAt
    }

    Comment {
        uuid id PK
        uuid companyId FK
        uuid threadId FK
        uuid authorUserId FK
        text body
        string visibilityScope
        datetime createdAt
        datetime editedAt
        datetime deletedAt
    }

    CommentMention {
        uuid id PK
        uuid commentId FK
        uuid mentionedUserId FK
        string notificationStatus
        datetime createdAt
    }

    UserNotification {
        uuid id PK
        uuid companyId FK
        uuid userId FK
        string type
        string sourceType
        uuid sourceId FK
        string status
        datetime createdAt
        datetime readAt
    }

    ActivityEvent {
        uuid id PK
        uuid companyId FK
        string contextType
        uuid contextId FK
        string eventType
        string actorType
        uuid actorUserId FK
        string visibilityScope
        text metadataJson
        datetime occurredAt
    }

    %% =========================
    %% Communication
    %% =========================
    Communication {
        uuid id PK
        uuid applicationId FK
        uuid candidateId FK
        uuid sentByUserId FK
        string channel
        string subject
        text body
        string status
        datetime scheduledAt
        datetime sentAt
        datetime createdAt
    }

    %% =========================
    %% Automation
    %% =========================
    AutomationRule {
        uuid id PK
        uuid companyId FK
        string name
        string description
        string scopeType
        uuid scopeId FK
        string triggerType
        string status
        uuid createdByUserId FK
        datetime createdAt
        datetime updatedAt
    }

    AutomationCondition {
        uuid id PK
        uuid automationRuleId FK
        int sequence
        string fieldPath
        string operator
        text valueJson
        string logicalJoin
    }

    AutomationAction {
        uuid id PK
        uuid automationRuleId FK
        int sequence
        string actionType
        text configJson
    }

    AutomationExecution {
        uuid id PK
        uuid companyId FK
        uuid automationRuleId FK
        uuid jobOpeningId FK
        uuid applicationId FK
        string triggeredByType
        uuid triggeredByUserId FK
        string triggerEventType
        string status
        string resultSummary
        datetime startedAt
        datetime finishedAt
    }

    AutomationExecutionItem {
        uuid id PK
        uuid automationExecutionId FK
        uuid automationActionId FK
        string status
        string message
        text payloadJson
        datetime executedAt
    }

    %% =========================
    %% AI Assistance
    %% =========================
    AiGenerationRun {
        uuid id PK
        uuid companyId FK
        string contextType
        uuid contextId FK
        string purpose
        string provider
        string modelVersion
        text promptText
        text responseText
        int promptTokens
        int completionTokens
        string status
        datetime startedAt
        datetime completedAt
    }

    AiSuggestion {
        uuid id PK
        uuid companyId FK
        uuid aiGenerationRunId FK
        string contextType
        uuid contextId FK
        string suggestionType
        string title
        text contentJson
        string status
        uuid acceptedByUserId FK
        datetime acceptedAt
        datetime createdAt
    }

    AiSuggestionFeedback {
        uuid id PK
        uuid aiSuggestionId FK
        uuid userId FK
        string decision
        text feedbackText
        datetime createdAt
    }

    %% =========================
    %% Relationships
    %% =========================
    Company ||--o{ User : crear
    Company ||--o{ Candidate : pertenecer
    Company ||--o{ JobOpening : contener
    Company ||--o{ PipelineTemplate : definir
    Company ||--o{ Application : contener
    Company ||--o{ SavedFilter : configurar
    Company ||--o{ CandidateViewEvent : registrar
    Company ||--o{ BulkOperation : agrupar
    Company ||--o{ SlaPolicy : definir
    Company ||--o{ SlaBreachEvent : registrar
    Company ||--o{ CommentThread : contener
    Company ||--o{ Comment : contener
    Company ||--o{ UserNotification : emitir
    Company ||--o{ ActivityEvent : registrar
    Company ||--o{ AutomationRule : definir
    Company ||--o{ AutomationExecution : registrar
    Company ||--o{ AiGenerationRun : ejecutar
    Company ||--o{ AiSuggestion : almacenar
    Company ||--o{ ScorecardTemplate : definir

    User ||--o{ JobOpening : crear
    User ||--o{ StageTransition : ejecutar
    User ||--o{ Evaluation : emitir
    User ||--o{ Note : redactar
    User ||--o{ Communication : enviar
    User ||--o{ SavedFilter : guardar
    User ||--o{ CandidateViewEvent : visualizar
    User ||--o{ BulkOperation : solicitar
    User ||--o{ CommentThread : iniciar
    User ||--o{ Comment : redactar
    User ||--o{ CommentMention : ser_mencionado
    User ||--o{ UserNotification : recibir
    User ||--o{ ActivityEvent : originar
    User ||--o{ AutomationRule : configurar
    User ||--o{ AutomationExecution : disparar
    User ||--o{ AiSuggestion : aceptar
    User ||--o{ AiSuggestionFeedback : responder
    User ||--o{ ScorecardTemplate : crear

    Candidate ||--o{ Application : postular
    Candidate ||--o{ Communication : recibir

    PipelineTemplate ||--o{ PipelineStage : definir

    JobOpening ||--o{ Application : recibir
    JobOpening ||--o{ PipelineStage : copiar
    JobOpening ||--o{ SlaPolicy : parametrizar
    JobOpening ||--o{ SlaBreachEvent : afectar
    JobOpening ||--o{ AutomationExecution : contextualizar

    PipelineStage ||--o{ Application : ubicar
    PipelineStage o|--o{ StageTransition : originar
    PipelineStage ||--o{ StageTransition : destinar
    PipelineStage o|--o{ Evaluation : contextualizar
    PipelineStage ||--o{ SlaPolicy : limitar
    PipelineStage ||--o{ SlaBreachEvent : incumplir

    Application ||--o{ StageTransition : registrar
    Application ||--o{ Evaluation : acumular
    Application ||--o{ Communication : registrar
    Application ||--o{ SlaBreachEvent : incumplir
    Application ||--o{ AutomationExecution : disparar

    BulkOperation ||--o{ BulkOperationItem : detallar

    ScorecardTemplate ||--o{ ScorecardCriterion : definir
    ScorecardTemplate ||--o{ Evaluation : estructurar

    Evaluation ||--o{ ScorecardResponse : responder
    ScorecardCriterion ||--o{ ScorecardResponse : evaluar

    CommentThread ||--o{ Comment : contener
    Comment ||--o{ CommentMention : mencionar

    AutomationRule ||--o{ AutomationCondition : evaluar
    AutomationRule ||--o{ AutomationAction : ejecutar
    AutomationRule ||--o{ AutomationExecution : disparar
    AutomationExecution ||--o{ AutomationExecutionItem : detallar
    AutomationAction ||--o{ AutomationExecutionItem : resultar

    SlaPolicy ||--o{ SlaBreachEvent : disparar

    AiGenerationRun ||--o{ AiSuggestion : producir
    AiSuggestion ||--o{ AiSuggestionFeedback : recibir


---

## Impacto en modelo existente

| Entidad existente | Tipo de impacto      | Descripción del cambio                                                                    | Requiere migración |
| ----------------- | -------------------- | ----------------------------------------------------------------------------------------- | ------------------ |
| Company           | Ninguno              | sin cambios estructurales; sigue como límite de tenant para nuevas entidades              | No                 |
| User              | Relacional indirecto | nuevas FKs desde filtros, lotes, comentarios, menciones, reglas, ejecuciones, sugerencias | No                 |
| Candidate         | Extensión menor      | completitud de perfil, resumen IA y marcas de visualización                               | Sí                 |
| JobOpening        | Extensión menor      | objetivo de tiempo y scorecard por defecto                                                | Sí                 |
| PipelineTemplate  | Ninguno              | sin cambios                                                                               | No                 |
| PipelineStage     | Extensión menor      | `slaHours` opcional                                                                       | Sí                 |
| Application       | Extensión media      | actividad, batch, SLA y campos denormalizados IA/automatización                           | Sí                 |
| StageTransition   | Ninguno              | se mantiene como fuente de verdad del pipeline                                            | No                 |
| Evaluation        | Extensión media      | scorecard estructurado y metadata de envío                                                | Sí                 |
| Note              | Ninguno              | se conserva como nota interna simple                                                      | No                 |
| Communication     | Ninguno              | sin cambios adicionales en esta extensión                                                 | No                 |

## Cierre

Con esta extensión, el modelo puede responder de forma directa:

* **Qué automatizaciones se dispararon para una vacante y con qué resultado** mediante `AutomationExecution` y `AutomationExecutionItem`.
* **Qué sugerencias IA se generaron y cuáles fueron aceptadas** mediante `AiSuggestion` y `AiSuggestionFeedback`.
* **Quién vio y comentó una postulación y cuándo** mediante `CandidateViewEvent`, `Comment`, `CommentMention` y `ActivityEvent`.
* **Qué métricas de eficiencia existen por vacante, recruiter y departamento** combinando `Application`, `BulkOperation`, `SlaBreachEvent`, `CandidateViewEvent` y timestamps denormalizados. Todo esto sigue respetando el modelo multi-tenant y evita romper los contratos base que ya definiste en el archivo.