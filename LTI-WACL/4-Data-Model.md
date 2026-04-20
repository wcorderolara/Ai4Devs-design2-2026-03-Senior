# Entidades principales del Modelo de Datos — ATS MVP

## 1. **Company**

* **Descripción:** Representa a la organización cliente que utiliza el ATS. Actúa como límite de tenant en un modelo SaaS B2B.
* **Atributos clave:**

  * `id: UUID`
  * `name: string`
  * `slug: string`
  * `status: enum(Active, Inactive)`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `Company 1 : N User`
  * `Company 1 : N JobOpening`
  * `Company 1 : N Candidate`
  * `Company 1 : N PipelineTemplate`
* **Reglas de negocio:**

  * El `slug` debe ser único por sistema.
  * Ninguna entidad operativa debe existir fuera de una `Company`.
  * Una `Company` inactiva no puede crear nuevas vacantes.

---

## 2. **User**

* **Descripción:** Representa a un usuario interno autenticado del ATS, como recruiter, admin o hiring manager. Participa en operación, evaluación y comunicación.
* **Atributos clave:**

  * `id: UUID`
  * `companyId: UUID`
  * `fullName: string`
  * `email: string`
  * `role: enum(Admin, Recruiter, HiringManager, Interviewer)`
  * `status: enum(Active, Inactive)`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `User N : 1 Company`
  * `User 1 : N JobOpening` (createdBy / ownedBy)
  * `User 1 : N Evaluation`
  * `User 1 : N Note`
  * `User 1 : N Communication`
* **Reglas de negocio:**

  * El `email` debe ser único dentro de una `Company`.
  * Solo usuarios activos pueden operar en el sistema.
  * El `role` determina permisos funcionales, no solo visibilidad.

---

## 3. **Candidate**

* **Descripción:** Representa a una persona postulante dentro del ATS. Es un Aggregate Root reutilizable entre múltiples vacantes.
* **Atributos clave:**

  * `id: UUID`
  * `companyId: UUID`
  * `firstName: string`
  * `lastName: string`
  * `email: string`
  * `phone: string?`
  * `currentLocation: string?`
  * `resumeUrl: string?`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `Candidate N : 1 Company`
  * `Candidate 1 : N Application`
  * `Candidate 1 : N Note`
  * `Candidate 1 : N Communication`
* **Reglas de negocio:**

  * Debe existir al menos un identificador de contacto principal, preferiblemente `email`.
  * Un candidato puede postular a múltiples vacantes.
  * El perfil del candidato no debe depender de una vacante específica.

---

## 4. **JobOpening**

* **Descripción:** Representa una vacante abierta o cerrada dentro de la empresa. Es el Aggregate Root del proceso de reclutamiento por posición.
* **Atributos clave:**

  * `id: UUID`
  * `companyId: UUID`
  * `title: string`
  * `department: string?`
  * `location: string?`
  * `workMode: enum(Onsite, Hybrid, Remote)?`
  * `description: text`
  * `status: enum(Draft, Open, Closed)`
  * `createdByUserId: UUID`
  * `createdAt: datetime`
  * `closedAt: datetime?`
* **Relaciones principales:**

  * `JobOpening N : 1 Company`
  * `JobOpening N : 1 User` (createdBy)
  * `JobOpening 1 : N Application`
  * `JobOpening 1 : 1 PipelineTemplate` o `N : 1 PipelineTemplate`
* **Reglas de negocio:**

  * Solo vacantes en estado `Open` aceptan nuevas postulaciones.
  * Una vacante cerrada conserva historial pero no admite cambios operativos críticos.
  * Toda vacante debe estar asociada a un pipeline válido.

---

## 5. **PipelineTemplate**

* **Descripción:** Define la estructura de etapas que usará una vacante. Permite estandarizar el flujo sin acoplarlo directamente a una sola postulación.
* **Atributos clave:**

  * `id: UUID`
  * `companyId: UUID`
  * `name: string`
  * `isDefault: boolean`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `PipelineTemplate N : 1 Company`
  * `PipelineTemplate 1 : N PipelineStage`
  * `PipelineTemplate 1 : N JobOpening`
* **Reglas de negocio:**

  * Una empresa puede tener varios templates, pero solo uno por defecto.
  * Un template debe tener al menos una etapa.
  * El orden de etapas debe ser determinístico.

---

## 6. **PipelineStage**

* **Descripción:** Representa una etapa definida dentro de un pipeline, como Applied, Screening o Interview. Funciona como entidad de referencia para ubicar cada aplicación.
* **Atributos clave:**

  * `id: UUID`
  * `pipelineTemplateId: UUID`
  * `name: string`
  * `sequence: integer`
  * `stageType: enum(Active, Rejected, Hired)`
* **Relaciones principales:**

  * `PipelineStage N : 1 PipelineTemplate`
  * `PipelineStage 1 : N Application`
* **Reglas de negocio:**

  * `sequence` debe ser único dentro de un `PipelineTemplate`.
  * Solo puede existir una etapa terminal de tipo `Hired` por pipeline.
  * Las etapas terminales no deben permitir movimiento posterior, salvo permiso explícito.

---

## 7. **Application**

* **Descripción:** Representa la postulación de un candidato a una vacante específica. Es la entidad central para responder en qué etapa está cada candidato por vacante.
* **Atributos clave:**

  * `id: UUID`
  * `companyId: UUID`
  * `candidateId: UUID`
  * `jobOpeningId: UUID`
  * `currentStageId: UUID`
  * `status: enum(Active, Rejected, Hired, Withdrawn)`
  * `appliedAt: datetime`
  * `source: string?`
  * `rejectionReason: string?`
* **Relaciones principales:**

  * `Application N : 1 Company`
  * `Application N : 1 Candidate`
  * `Application N : 1 JobOpening`
  * `Application N : 1 PipelineStage` (currentStage)
  * `Application 1 : N StageTransition`
  * `Application 1 : N Evaluation`
  * `Application 1 : N Note`
  * `Application 1 : N Communication`
* **Reglas de negocio:**

  * Debe existir una sola `Application` activa por combinación `Candidate + JobOpening`.
  * Toda aplicación debe tener una etapa actual válida.
  * El `status` debe ser consistente con la naturaleza de `currentStageId`.

---

## 8. **StageTransition**

* **Descripción:** Registra el historial de movimientos de una aplicación entre etapas. Permite trazabilidad operativa y auditoría básica del pipeline.
* **Atributos clave:**

  * `id: UUID`
  * `applicationId: UUID`
  * `fromStageId: UUID?`
  * `toStageId: UUID`
  * `changedByUserId: UUID`
  * `changedAt: datetime`
  * `comment: string?`
* **Relaciones principales:**

  * `StageTransition N : 1 Application`
  * `StageTransition N : 1 User` (changedBy)
  * `StageTransition N : 1 PipelineStage` (fromStage)
  * `StageTransition N : 1 PipelineStage` (toStage)
* **Reglas de negocio:**

  * Toda transición debe apuntar a una etapa perteneciente al pipeline de la vacante.
  * La primera transición puede tener `fromStageId = null`.
  * `changedAt` debe reflejar orden cronológico consistente por aplicación.

---

## 9. **Evaluation**

* **Descripción:** Representa una valoración formal de un candidato dentro de una postulación. Captura quién evaluó, cuándo y con qué resultado.
* **Atributos clave:**

  * `id: UUID`
  * `applicationId: UUID`
  * `evaluatorUserId: UUID`
  * `stageId: UUID?`
  * `score: decimal?`
  * `recommendation: enum(StrongNo, No, Yes, StrongYes)`
  * `summary: text?`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `Evaluation N : 1 Application`
  * `Evaluation N : 1 User` (evaluator)
  * `Evaluation N : 1 PipelineStage`
* **Reglas de negocio:**

  * Solo usuarios internos autorizados pueden emitir evaluaciones.
  * Una evaluación debe asociarse a una aplicación existente.
  * `recommendation` no puede quedar vacía si la evaluación se marca como completada.

---

## 10. **Note**

* **Descripción:** Representa un comentario interno no visible al candidato. Sirve para colaboración contextual durante el proceso.
* **Atributos clave:**

  * `id: UUID`
  * `applicationId: UUID?`
  * `candidateId: UUID?`
  * `authorUserId: UUID`
  * `content: text`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `Note N : 1 User` (author)
  * `Note N : 1 Candidate`
  * `Note N : 1 Application`
* **Reglas de negocio:**

  * Una nota debe asociarse al menos a `Candidate` o `Application`.
  * Las notas son siempre internas.
  * No deben eliminarse físicamente si afectan trazabilidad; preferible soft delete futuro.

---

## 11. **Communication**

* **Descripción:** Registra cada comunicación enviada al candidato desde el ATS. Permite responder qué mensajes se enviaron y cuándo.
* **Atributos clave:**

  * `id: UUID`
  * `applicationId: UUID`
  * `candidateId: UUID`
  * `sentByUserId: UUID`
  * `channel: enum(Email)`
  * `subject: string`
  * `body: text`
  * `status: enum(Pending, Sent, Failed)`
  * `sentAt: datetime?`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `Communication N : 1 Application`
  * `Communication N : 1 Candidate`
  * `Communication N : 1 User` (sentBy)
* **Reglas de negocio:**

  * Toda comunicación debe estar vinculada a una aplicación.
  * Solo comunicaciones con `status = Sent` deben tener `sentAt`.
  * El candidato asociado debe coincidir con el candidato de la aplicación.

---

## 12. **Role**

* **Descripción:** Representa un rol de autorización del sistema. En MVP puede operar como catálogo simple para controlar acceso funcional.
* **Atributos clave:**

  * `id: UUID`
  * `name: string`
  * `code: string`
  * `createdAt: datetime`
* **Relaciones principales:**

  * `Role 1 : N User`
* **Reglas de negocio:**

  * `code` debe ser único por sistema o por tenant, según estrategia.
  * Un usuario debe tener un único rol principal en MVP.
  * Los permisos efectivos derivan del rol asignado.

---

# Value Objects sugeridos

## 13. **PersonName** *(Value Object)*

* **Descripción:** Encapsula la representación del nombre de una persona. Evita dispersar reglas de formato en múltiples entidades.
* **Atributos clave:**

  * `firstName: string`
  * `lastName: string`
* **Relaciones principales:**

  * Embebido en `Candidate`
  * Embebido en `User`
* **Reglas de negocio:**

  * No debe existir vacío en ambas partes.
  * Debe persistirse normalizado para búsqueda y visualización.

## 14. **EmailAddress** *(Value Object)*

* **Descripción:** Representa una dirección de correo validada. Centraliza reglas de formato y comparación.
* **Atributos clave:**

  * `value: string`
* **Relaciones principales:**

  * Embebido en `Candidate`
  * Embebido en `User`
* **Reglas de negocio:**

  * Debe almacenarse en formato normalizado.
  * Debe cumplir validación sintáctica antes de persistencia.

---

# Aggregate Roots recomendados

* **Company** → raíz del tenant
* **JobOpening** → raíz del proceso por vacante
* **Candidate** → raíz del perfil de postulante
* **Application** → raíz transaccional del vínculo Candidate-Vacante

---

# Decisiones de modelado clave

## 1. Etapa por candidato y vacante

La pregunta **“¿En qué etapa está cada candidato por vacante?”** se responde desde `Application.currentStageId`, respaldado por `StageTransition` para historial.

## 2. Quién evaluó a quién y con qué resultado

La pregunta **“¿Quién evaluó a quién y con qué resultado?”** se responde desde `Evaluation`, relacionando:

* `evaluatorUserId`
* `applicationId`
* `score`
* `recommendation`

## 3. Qué comunicaciones se enviaron y cuándo

La pregunta **“¿Qué comunicaciones se enviaron y cuándo?”** se responde desde `Communication`, usando:

* `applicationId`
* `candidateId`
* `channel`
* `status`
* `sentAt`

---

# Alcance MVP mínimo recomendado

## Core obligatorio

* Company
* User
* Role
* Candidate
* JobOpening
* PipelineTemplate
* PipelineStage
* Application
* StageTransition
* Evaluation
* Note
* Communication

## Value Objects útiles desde el inicio

* PersonName
* EmailAddress


# Resultado de Claude Sonnet 4.6
Aquí el modelo de datos completo con todos los cambios aplicados y explicados:

---

# Modelo de Datos — ATS MVP

## Cambios aplicados al modelo original

Antes de las entidades, un resumen de las 5 modificaciones incorporadas:

**Cambio 1 — `Application.currentStageId` desnormalizado con regla explícita**
El campo existía pero sin ninguna regla de sincronización documentada. Se añade `stageUpdatedAt` y se establece como invariante que `currentStageId` debe actualizarse en la misma transacción que el `INSERT` a `StageTransition`. Esto evita desincronización en escrituras concurrentes.

**Cambio 2 — `Note` sin polimorfismo implícito**
El modelo original tenía `applicationId?: UUID` y `candidateId?: UUID` con la regla "al menos uno no null", lo cual no es constrainteable en base de datos relacional. Se reemplaza por `contextType: enum(Application, Candidate)` + `contextId: UUID`, patrón polimórfico explícito que sí admite `NOT NULL` y permite índices eficientes.

**Cambio 3 — Entidad `Role` eliminada, consolidada en enum de `User`**
El modelo tenía `User.role: enum` y además una entidad `Role` separada con su propia tabla, lo cual era una contradicción. Para MVP el enum es suficiente. La entidad `Role` queda documentada como deuda técnica para cuando se necesiten permisos granulares dinámicos.

**Cambio 4 — `Communication` con `scheduledAt` y alcance de canal documentado**
Se añade `scheduledAt: datetime?` para distinguir entre envío inmediato y programado. Se documenta explícitamente que `channel: enum(Email)` solo implementa `Email` en MVP, bloqueando sobrediseño prematuro en la capa de envío.

**Cambio 5 — `PipelineTemplate` con patrón snapshot al crear vacante**
El modelo original referenciaba el template vivo desde `JobOpening`, lo que significa que modificar un template afectaría vacantes ya abiertas. Se introduce el patrón snapshot: al crear una `JobOpening`, las etapas del template se copian como registros independientes con `jobOpeningId`. El campo `sourceTemplateId` queda solo como referencia de origen, sin acoplamiento operativo.

---

## Entidades

---

### 1. Company

Representa a la organización cliente que utiliza el ATS. Actúa como límite de tenant en un modelo SaaS B2B.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `name` | string | |
| `slug` | string | único por sistema |
| `status` | enum(Active, Inactive) | |
| `createdAt` | datetime | |

**Relaciones**

- `Company 1 : N User`
- `Company 1 : N JobOpening`
- `Company 1 : N Candidate`
- `Company 1 : N PipelineTemplate`

**Reglas de negocio**

- El `slug` debe ser único por sistema.
- Ninguna entidad operativa puede existir fuera de una `Company`.
- Una `Company` inactiva no puede crear nuevas vacantes.

**Clasificación DDD:** Aggregate Root (raíz del tenant)

---

### 2. User

Representa a un usuario interno autenticado del ATS. Participa en operación, evaluación y comunicación.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `companyId` | UUID | FK → Company |
| `fullName` | string | Ver Value Object `PersonName` |
| `email` | string | Ver Value Object `EmailAddress` |
| `role` | enum(Admin, Recruiter, HiringManager, Interviewer) | ⚠️ Cambio 3: único campo de rol, entidad `Role` eliminada en MVP |
| `status` | enum(Active, Inactive) | |
| `createdAt` | datetime | |

**Relaciones**

- `User N : 1 Company`
- `User 1 : N JobOpening` (createdBy)
- `User 1 : N Evaluation`
- `User 1 : N Note`
- `User 1 : N Communication`

**Reglas de negocio**

- El `email` debe ser único dentro de una `Company`.
- Solo usuarios con `status = Active` pueden operar en el sistema.
- El `role` determina permisos funcionales, no solo visibilidad.

**Deuda técnica documentada:** La entidad `Role` separada tiene sentido cuando se necesiten permisos granulares dinámicos (post-MVP).

**Clasificación DDD:** Entity

---

### 3. Candidate

Representa a una persona postulante. Es un Aggregate Root reutilizable entre múltiples vacantes.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `companyId` | UUID | FK → Company |
| `firstName` | string | Ver Value Object `PersonName` |
| `lastName` | string | Ver Value Object `PersonName` |
| `email` | string | Ver Value Object `EmailAddress` |
| `phone` | string? | opcional |
| `currentLocation` | string? | opcional |
| `resumeUrl` | string? | opcional |
| `createdAt` | datetime | |

**Relaciones**

- `Candidate N : 1 Company`
- `Candidate 1 : N Application`
- `Candidate 1 : N Note`
- `Candidate 1 : N Communication`

**Reglas de negocio**

- Debe existir al menos un identificador de contacto principal, preferiblemente `email`.
- Un candidato puede postular a múltiples vacantes.
- El perfil no debe depender de una vacante específica.

**Clasificación DDD:** Aggregate Root

---

### 4. JobOpening

Representa una vacante abierta o cerrada. Es el Aggregate Root del proceso de reclutamiento por posición.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `companyId` | UUID | FK → Company |
| `title` | string | |
| `department` | string? | opcional |
| `location` | string? | opcional |
| `workMode` | enum(Onsite, Hybrid, Remote)? | opcional |
| `description` | text | |
| `status` | enum(Draft, Open, Closed) | |
| `sourceTemplateId` | UUID? | ⚠️ Cambio 5: renombrado de `pipelineTemplateId`. Solo referencia de origen, sin acoplamiento operativo |
| `createdByUserId` | UUID | FK → User |
| `createdAt` | datetime | |
| `closedAt` | datetime? | opcional |

**Relaciones**

- `JobOpening N : 1 Company`
- `JobOpening N : 1 User` (createdBy)
- `JobOpening 1 : N Application`
- `JobOpening 1 : N PipelineStage` ⚠️ Cambio 5: las etapas se copian directamente a la vacante en el momento de creación

**Reglas de negocio**

- Solo vacantes con `status = Open` aceptan nuevas postulaciones.
- Al crear una vacante, se ejecuta un snapshot de las etapas del `PipelineTemplate` seleccionado: se hace `INSERT` de cada `PipelineStage` con `jobOpeningId = this.id`. Desde ese momento, las etapas son independientes del template original.
- Una vacante cerrada conserva historial pero no admite cambios operativos críticos.

**Clasificación DDD:** Aggregate Root

---

### 5. PipelineTemplate

Define la estructura de etapas reutilizable. Sirve como punto de partida para nuevas vacantes, pero no controla las etapas de vacantes ya creadas.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `companyId` | UUID | FK → Company |
| `name` | string | |
| `isDefault` | boolean | |
| `createdAt` | datetime | |

**Relaciones**

- `PipelineTemplate N : 1 Company`
- `PipelineTemplate 1 : N PipelineStage` (etapas del template, no de la vacante)

**Reglas de negocio**

- Una empresa puede tener varios templates pero solo uno `isDefault = true`.
- Un template debe tener al menos una etapa.
- El orden de etapas debe ser determinístico (`sequence`).
- ⚠️ Cambio 5: modificar un template no afecta vacantes ya abiertas. Las etapas de las vacantes son copias independientes.

**Clasificación DDD:** Entity

---

### 6. PipelineStage

Representa una etapa dentro de un pipeline. En el modelo anterior solo existía vinculada al template. Con el cambio 5, esta entidad sirve tanto para templates como para vacantes concretas, distinguidas por `ownerType`.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `ownerType` | enum(Template, JobOpening) | ⚠️ Cambio 5: distingue si la etapa pertenece a un template o a una vacante concreta |
| `ownerId` | UUID | FK → `PipelineTemplate.id` o `JobOpening.id` según `ownerType` |
| `name` | string | |
| `sequence` | integer | orden dentro del pipeline |
| `stageType` | enum(Active, Rejected, Hired) | |

**Relaciones**

- `PipelineStage N : 1 PipelineTemplate` (cuando `ownerType = Template`)
- `PipelineStage N : 1 JobOpening` (cuando `ownerType = JobOpening`)
- `PipelineStage 1 : N Application` (solo las de tipo JobOpening)

**Reglas de negocio**

- `sequence` debe ser único dentro del mismo `ownerId`.
- Solo puede existir una etapa `stageType = Hired` por pipeline.
- Las etapas terminales (`Hired`, `Rejected`) no permiten movimiento posterior salvo permiso explícito.
- Solo las etapas con `ownerType = JobOpening` pueden ser referenciadas por `Application.currentStageId`.

**Clasificación DDD:** Entity

---

### 7. Application

Representa la postulación de un candidato a una vacante. Es la entidad transaccional central del sistema.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `companyId` | UUID | FK → Company |
| `candidateId` | UUID | FK → Candidate |
| `jobOpeningId` | UUID | FK → JobOpening |
| `currentStageId` | UUID | ⚠️ Cambio 1: campo desnormalizado para lectura rápida. Fuente de verdad = último `StageTransition` |
| `stageUpdatedAt` | datetime | ⚠️ Cambio 1: campo nuevo. Se actualiza en la misma tx que `StageTransition` |
| `status` | enum(Active, Rejected, Hired, Withdrawn) | |
| `appliedAt` | datetime | |
| `source` | string? | canal de origen |
| `rejectionReason` | string? | opcional |

**Relaciones**

- `Application N : 1 Company`
- `Application N : 1 Candidate`
- `Application N : 1 JobOpening`
- `Application N : 1 PipelineStage` (currentStage)
- `Application 1 : N StageTransition`
- `Application 1 : N Evaluation`
- `Application 1 : N Note`
- `Application 1 : N Communication`

**Reglas de negocio**

- Debe existir una sola `Application` activa por combinación `Candidate + JobOpening`.
- ⚠️ Cambio 1: `currentStageId` y `stageUpdatedAt` deben actualizarse atómicamente en la misma transacción que el `INSERT` a `StageTransition`. Nunca actualizar uno sin el otro.
- El `status` debe ser consistente con el `stageType` de `currentStageId`.

**Clasificación DDD:** Aggregate Root (raíz transaccional del vínculo Candidate-Vacante)

---

### 8. StageTransition

Registra el historial de movimientos de una aplicación entre etapas. Fuente de verdad del pipeline.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `applicationId` | UUID | FK → Application |
| `fromStageId` | UUID? | null en la primera transición |
| `toStageId` | UUID | FK → PipelineStage |
| `changedByUserId` | UUID | FK → User |
| `changedAt` | datetime | |
| `comment` | string? | opcional |

**Relaciones**

- `StageTransition N : 1 Application`
- `StageTransition N : 1 User` (changedBy)
- `StageTransition N : 1 PipelineStage` (fromStage)
- `StageTransition N : 1 PipelineStage` (toStage)

**Reglas de negocio**

- Toda transición debe apuntar a una etapa perteneciente al `JobOpening` de la aplicación (no a etapas de template).
- La primera transición puede tener `fromStageId = null`.
- `changedAt` debe mantener orden cronológico consistente por aplicación.
- ⚠️ Cambio 1: todo `INSERT` en esta tabla debe ir acompañado de un `UPDATE` a `Application.currentStageId` y `Application.stageUpdatedAt` en la misma transacción.

**Clasificación DDD:** Entity

---

### 9. Evaluation

Representa una valoración formal de un candidato dentro de una postulación.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `applicationId` | UUID | FK → Application |
| `evaluatorUserId` | UUID | FK → User |
| `stageId` | UUID? | etapa en que se realizó la evaluación |
| `score` | decimal? | opcional |
| `recommendation` | enum(StrongNo, No, Yes, StrongYes) | obligatorio si `completedAt` no es null |
| `summary` | text? | opcional |
| `createdAt` | datetime | |
| `completedAt` | datetime? | null = evaluación en borrador |

**Relaciones**

- `Evaluation N : 1 Application`
- `Evaluation N : 1 User` (evaluator)
- `Evaluation N : 1 PipelineStage`

**Reglas de negocio**

- Solo usuarios internos autorizados pueden emitir evaluaciones.
- `recommendation` no puede ser null si `completedAt` tiene valor.
- Una evaluación con `completedAt = null` se considera borrador y no cuenta para decisiones.

**Clasificación DDD:** Entity

---

### 10. Note

Representa un comentario interno no visible al candidato.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `contextType` | enum(Application, Candidate) | ⚠️ Cambio 2: reemplaza los dos FK opcionales del modelo original |
| `contextId` | UUID | ⚠️ Cambio 2: FK hacia `Application.id` o `Candidate.id` según `contextType`. Siempre NOT NULL |
| `authorUserId` | UUID | FK → User |
| `content` | text | |
| `createdAt` | datetime | |

**Relaciones**

- `Note N : 1 User` (author)
- `Note N : 1 Candidate` (cuando `contextType = Candidate`)
- `Note N : 1 Application` (cuando `contextType = Application`)

**Reglas de negocio**

- `contextId` es siempre `NOT NULL`. El tipo de contexto lo determina `contextType`.
- Las notas son siempre internas; nunca visibles al candidato.
- No se eliminan físicamente; se recomienda soft delete para preservar trazabilidad.

**Por qué se cambió:** el patrón anterior con dos FK opcionales no puede expresarse como `NOT NULL` en base de datos, lo que obliga a validar la regla en código. El patrón `contextType + contextId` es un polimorfismo explícito que sí es constrainteable y más eficiente de indexar.

**Clasificación DDD:** Entity

---

### 11. Communication

Registra cada comunicación enviada al candidato desde el ATS.

| Atributo | Tipo | Notas |
|---|---|---|
| `id` | UUID | PK |
| `applicationId` | UUID | FK → Application |
| `candidateId` | UUID | FK → Candidate |
| `sentByUserId` | UUID | FK → User |
| `channel` | enum(Email) | ⚠️ Cambio 4: MVP implementa solo Email. Enum preparado para extensión futura |
| `subject` | string | |
| `body` | text | |
| `status` | enum(Pending, Sent, Failed) | |
| `scheduledAt` | datetime? | ⚠️ Cambio 4: campo nuevo. Distingue envío inmediato de programado |
| `sentAt` | datetime? | null hasta que `status = Sent` |
| `createdAt` | datetime | |

**Relaciones**

- `Communication N : 1 Application`
- `Communication N : 1 Candidate`
- `Communication N : 1 User` (sentBy)

**Reglas de negocio**

- Toda comunicación debe estar vinculada a una aplicación.
- Solo comunicaciones con `status = Sent` deben tener `sentAt` con valor.
- Si `scheduledAt` tiene valor y `sentAt` es null, la comunicación está pendiente de envío programado.
- El `candidateId` debe coincidir con el candidato de la `Application` referenciada.
- ⚠️ Cambio 4: no implementar canales distintos a `Email` en MVP aunque el enum los permita en el futuro. La capa de envío solo necesita un proveedor de email.

**Clasificación DDD:** Entity

---

## Value Objects

### PersonName

Encapsula la representación del nombre de una persona.

| Atributo | Tipo |
|---|---|
| `firstName` | string |
| `lastName` | string |

Embebido en `Candidate` y `User`. Ninguna de las dos partes puede estar vacía. Se normaliza para búsqueda y visualización.

---

### EmailAddress

Representa una dirección de correo validada.

| Atributo | Tipo |
|---|---|
| `value` | string |

Embebido en `Candidate` y `User`. Se almacena normalizado (lowercase). Debe pasar validación sintáctica antes de persistencia.

---

## Aggregate Roots

| Entidad | Rol |
|---|---|
| `Company` | Raíz del tenant |
| `JobOpening` | Raíz del proceso por vacante |
| `Candidate` | Raíz del perfil de postulante |
| `Application` | Raíz transaccional del vínculo Candidate-Vacante |

---

## Criterios de éxito del modelo

**¿En qué etapa está cada candidato por vacante?**
`Application.currentStageId` → `PipelineStage.name`. Historial completo en `StageTransition`.

**¿Quién evaluó a quién y con qué resultado?**
`Evaluation.evaluatorUserId` + `Evaluation.applicationId` + `Evaluation.recommendation` + `Evaluation.score`.

**¿Qué comunicaciones se enviaron y cuándo?**
`Communication` filtrado por `applicationId` o `candidateId`, ordenado por `sentAt`. Estado en `Communication.status`.

---

## Alcance MVP

### Core obligatorio
`Company`, `User`, `Candidate`, `JobOpening`, `PipelineTemplate`, `PipelineStage`, `Application`, `StageTransition`, `Evaluation`, `Note`, `Communication`

### Eliminado del MVP
`Role` como entidad separada — reemplazada por el enum en `User`.

### Value Objects útiles desde el inicio
`PersonName`, `EmailAddress`