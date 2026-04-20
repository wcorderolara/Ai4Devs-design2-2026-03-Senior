# 1: Initial Step (Gather information)

Eres un experto en Diseño y Desarrollo de productos Digitales, tienes experiencia en el domitio de plataformas ATS (Applicant-Tracking Systems).

- ¿Cuáles son los beneficios que un sistema ATS le da a una empresa?
- ¿Cuáles son las principales funcionalidades que debería tener un sistema ATS?, describe las más importantes ordenadas por prioridad de mayor a menor prioridad.
- ¿Cuál seria el perfil ideal de una empresa para utilizar un ATS?, conocer ¿Qué le duele? ¿Qué quiere alcanzar? ¿Cuáles son los mayores temores? y ¿Qué ha intentando antes de usar un ATS?, de esta ultima dame entre 3 y 5 puntos principales separandolas en listas

# Brainstorming
### Funcionalidades Principales
Cuales serían los funcionalidades básicas mínimas requeridas debe tener un Sistema ATS, dame el resultado en un listado con nombre de funcionalidad, Definición o Descripción y beneficio que da al customer , no agregues mas documentacion de la necesaria ni utilices verbosidad, tampoco definas siguientes pasos, iremos trabajando poco a poco en ello.

### Customer Journey
¿Cómo seria el Customer Journey de un customer para utilizar un Applicant-Tracking System?, describeme los pasos de todas las iteraciones y posibles relaciones

# Use Cases (Definition)
### Rol y Contexto
Eres un Arquitecto de Software Senior con más de 10 años de experiencia diseñando 
sistemas empresariales de recursos humanos. Has liderado implementaciones de ATS 
(Applicant Tracking Systems) para empresas medianas y grandes, y conoces a fondo 
los estándares de la industria (SHRM, LinkedIn Talent Solutions, Greenhouse, Lever).

### Tarea
Analiza y define los casos de uso esenciales para la fase MVP de un ATS, priorizando 
funcionalidad sobre complejidad técnica.

### Instrucciones de formato
Para cada caso de uso, proporciona:
1. **Nombre** del caso de uso (verbo + objeto, ej. "Publicar Vacante")
2. **Actor(es)** involucrados
3. **Descripción** breve (2-3 oraciones)
4. **Precondiciones** necesarias
5. **Resultado esperado**
6. **Prioridad** (Alta / Media)

### Restricciones
- Limítate a los casos de uso necesarios para una funcionalidad básica (MVP)
- Agrupa los casos de uso por módulo funcional
- No incluyas integraciones externas complejas en esta fase
- Usa lenguaje técnico preciso, apropiado para un equipo de desarrollo

### Audiencia
El output será consumido por desarrolladores fullstack y el Product Owner 
para comenzar el backlog del proyecto.

# Data Model
### Rol y Contexto
Eres un Product Designer Senior con especialización en sistemas de RRHH digitales,
combinado con la experiencia de un Arquitecto de Software con más de 10 años
diseñando modelos de datos para plataformas SaaS B2B. Has trabajado en productos
como Greenhouse, Lever y Workday, y conoces los patrones de dominio propios
de un ATS (Applicant Tracking System).

### Tarea
Define las entidades principales del Modelo de Datos para la fase MVP de un ATS,
priorizando claridad del dominio y extensibilidad futura sobre optimización técnica
prematura.

### Instrucciones de formato
Para cada entidad proporciona:
1. **Nombre** de la entidad (sustantivo en singular, PascalCase)
2. **Descripción** del propósito dentro del dominio (2 oraciones máximo)
3. **Atributos clave** (solo los esenciales para MVP, con tipo de dato)
4. **Relaciones principales** con otras entidades (cardinalidad incluida)
5. **Reglas de negocio** relevantes (1–3 restricciones o invariantes del dominio)

### Restricciones
- Limítate a entidades del core domain: no incluyas módulos de nómina,
  onboarding ni integraciones externas en esta fase
- Usa nomenclatura de Domain-Driven Design (Aggregate Root, Entity, Value Object)
  donde aplique
- No generes el DDL ni el diagrama ER todavía; enfócate en la definición semántica
- El modelo debe soportar los casos de uso ya definidos:
  Gestión de Vacantes, Candidatos, Pipeline, Evaluación, Comunicación
  y Administración

### Audiencia
El output será revisado por el Tech Lead y el Product Owner para validar
el dominio antes de pasar al diseño físico de la base de datos.

### Criterio de éxito
El modelo debe permitir responder: ¿En qué etapa está cada candidato
por vacante? ¿Quién evaluó a quién y con qué resultado?
¿Qué comunicaciones se enviaron y cuándo?

# ERD - Claude and ChatGPT
## Rol y Contexto
Eres un Arquitecto de Software Senior especializado en modelado de datos para
plataformas SaaS B2B. Conoces a fondo la sintaxis de Mermaid erDiagram y las
convenciones de notación crow's foot para cardinalidad.

## Tarea
Genera el diagrama entidad-relación en sintaxis Mermaid (erDiagram) para el
modelo de datos del ATS MVP descrito en el contexto, incorporando todos los
cambios aplicados al modelo original.

## Contexto del modelo
[PEGAR AQUÍ EL MODELO DE DATOS COMPLETO EN MARKDOWN]

## Instrucciones de formato
- Usa sintaxis erDiagram de Mermaid exclusivamente
- Representa todas las entidades con sus atributos clave (tipo + nombre)
- Usa notación crow's foot para cardinalidad: ||, |{, o|, etc.
- Incluye el label de relación como verbo en infinitivo entre comillas
- Marca PKs y FKs explícitamente con los comentarios PK y FK
- Agrupa visualmente las entidades por módulo usando comentarios %% 
- No incluyas Value Objects como entidades separadas; son campos embebidos
- No incluyas la entidad Role

## Restricciones
- Solo sintaxis válida para Mermaid erDiagram, sin extensiones
- Los nombres de atributos deben ser camelCase sin espacios
- Los tipos de datos deben ser simples: string, uuid, int, datetime, boolean, text, decimal
- No uses comillas en nombres de entidades ni atributos
- El diagrama debe poder renderizarse sin errores en mermaid.live

## Criterio de éxito
El diagrama debe responder visualmente las tres preguntas clave del modelo:
1. En que etapa esta cada candidato por vacante
2. Quien evaluo a quien y con que resultado
3. Que comunicaciones se enviaron y cuando

# HIGH LEVEL DESIGN & C4 DIAGRAM
## Rol y Contexto
Eres un Arquitecto de Software Senior con experiencia diseñando sistemas SaaS B2B
de escala media. Has documentado arquitecturas para equipos de desarrollo usando
diagramas C4, diagramas de componentes y diagramas de secuencia. Conoces los
patrones comunes de plataformas de reclutamiento como Greenhouse y Lever.

## Tarea
Genera el diseño del sistema a alto nivel para el ATS MVP, explicando cada
decisión arquitectónica con su diagrama correspondiente. El output debe ser
suficiente para que un equipo de desarrollo entienda cómo construir el sistema
sin ambigüedad.

## Contexto del sistema
- Plataforma: SaaS B2B multi-tenant
- Stack definido: Angular (frontend), Node.js/Express (backend),
  PostgreSQL (base de datos), AWS (infraestructura)
- Usuarios concurrentes esperados en MVP: bajo (< 500 por tenant)
- Modelo de datos: 
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

- Casos de uso: ### Gestión de Vacantes
flowchart LR
    %% Módulo: Gestión de Vacantes
    Recruiter[Recruiter]
    Admin[Admin]

    subgraph Gestion_de_Vacantes
        UC1((Crear Vacante))
        UC2((Editar Vacante))
        UC3((Cerrar Vacante))
    end

    Recruiter --> UC1
    Recruiter --> UC2
    Recruiter --> UC3

    Admin --> UC1
    Admin --> UC2
    Admin --> UC3

### Captura de Datos
flowchart LR
    %% Módulo: Captura de Candidatos
    Candidato[Candidato]
    Recruiter[Recruiter]
    Admin[Admin]

    subgraph Captura_de_Candidatos
        UC4((Postular a Vacante))
        UC5((Registrar Candidato Manualmente))
    end

    Candidato --> UC4
    Recruiter --> UC5
    Admin --> UC5

### Gestión de Candidatos
flowchart LR
    %% Módulo: Gestión de Candidatos
    Recruiter[Recruiter]
    HiringManager[Hiring Manager]
    Admin[Admin]

    subgraph Gestion_de_Candidatos
        UC6((Consultar Perfil de Candidato))
        UC7((Buscar Candidato))
        UC8((Actualizar Información de Candidato))
    end

    Recruiter --> UC6
    Recruiter --> UC7
    Recruiter --> UC8

    HiringManager --> UC6
    HiringManager --> UC7

    Admin --> UC6
    Admin --> UC7
    Admin --> UC8

### Pipeline de Reclutamiento
flowchart LR
    %% Módulo: Gestión de Candidatos
    Recruiter[Recruiter]
    HiringManager[Hiring Manager]
    Admin[Admin]

    subgraph Gestion_de_Candidatos
        UC6((Consultar Perfil de Candidato))
        UC7((Buscar Candidato))
        UC8((Actualizar Información de Candidato))
    end

    Recruiter --> UC6
    Recruiter --> UC7
    Recruiter --> UC8

    HiringManager --> UC6
    HiringManager --> UC7

    Admin --> UC6
    Admin --> UC7
    Admin --> UC8

### Evaluación y Colaboración
flowchart LR
    %% Módulo: Evaluación y Colaboración
    Recruiter[Recruiter]
    HiringManager[Hiring Manager]
    Interviewer[Interviewer]

    subgraph Evaluacion_y_Colaboracion
        UC13((Registrar Feedback de Candidato))
        UC14((Registrar Evaluación de Entrevista))
    end

    Recruiter --> UC13
    Recruiter --> UC14

    HiringManager --> UC13
    HiringManager --> UC14

    Interviewer --> UC13
    Interviewer --> UC14

### Comunicación Operativa
flowchart LR
    %% Módulo: Comunicación Operativa
    Recruiter[Recruiter]

    subgraph Comunicacion_Operativa
        UC15((Enviar Comunicación al Candidato))
    end

    Recruiter --> UC15

### Administración Básica
flowchart LR
    %% Módulo: Administración Básica
    Admin[Admin]

    subgraph Administracion_Basica
        UC16((Gestionar Usuarios))
        UC17((Asignar Roles y Permisos Básicos))
    end

    Admin --> UC16
    Admin --> UC17

## Diagramas requeridos
Genera los siguientes diagramas en sintaxis Mermaid, en este orden:

1. **Diagrama de contexto (C4 nivel 1)**
   - El sistema ATS como caja central
   - Actores externos: Recruiter, Candidate, Admin, Email Provider
   - Relaciones entre actores y el sistema

2. **Diagrama de contenedores (C4 nivel 2)**
   - Frontend SPA (Angular)
   - Backend API REST (Node.js/Express)
   - Base de datos (PostgreSQL)
   - Servicio de email
   - Relaciones y protocolos entre contenedores (HTTP, SQL, SMTP)

3. **Diagrama de componentes del backend (C4 nivel 3)**
   - Módulos internos: JobOpeningModule, CandidateModule, PipelineModule,
     EvaluationModule, CommunicationModule, AuthModule
   - Dependencias entre módulos
   - Relación con la base de datos

4. **Diagrama de secuencia — flujo principal**
   Modela el flujo: Recruiter crea vacante → Candidato postula →
   Reclutador mueve candidato de etapa → Sistema envía comunicación

5. **Diagrama de flujo — ciclo de vida de una Application**
   Estados: Draft → Open → (candidato postula) → Active →
   (decisión) → Hired | Rejected | Withdrawn

## Instrucciones de formato
Para cada diagrama:
- Título en comentario %% antes del bloque
- Sintaxis Mermaid válida para mermaid.live
- Después del bloque de código: párrafo de explicación de 3 a 5 oraciones
  justificando las decisiones arquitectónicas representadas

## Restricciones
- No uses PlantUML ni otro lenguaje que no sea Mermaid
- No generes código de implementación, solo arquitectura
- Mantén cada diagrama enfocado: máximo 10 nodos por diagrama
- Las decisiones deben ser coherentes con el modelo de datos ya definido
- No incluyas integraciones externas fuera del scope MVP (no SSO, no webhooks)

## Audiencia
El output será presentado al equipo técnico completo (frontend, backend, DevOps)
y al Product Owner en una sesión de kickoff. Debe ser comprensible para todos
los perfiles sin perder precisión técnica.

## Criterio de éxito
Al terminar de leer el documento, cualquier developer del equipo debe poder
responder: donde vive cada pieza del sistema, como se comunican entre si,
y cual es el flujo de datos del caso de uso principal.

# Charts
### Lean Canvas
Crea el diseño de un Diagrama Lean Canvas sobre las funcionalidades que hemos discutido para el ATS, debe poder ser creado en mermaid.

### CJM
Elabora el CJM del Customer Journey del usuario que implementa el ATS, debe ser ejecutado para mermaid