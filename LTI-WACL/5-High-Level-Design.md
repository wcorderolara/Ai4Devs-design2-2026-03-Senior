Aquí tienes el **diseño del sistema ATS MVP a alto nivel**, siguiendo exactamente los lineamientos definidos y alineado con el modelo de datos proporcionado .

---

# 1. Diagrama de Contexto (C4 Nivel 1)

```mermaid
%% C4 Nivel 1 - Diagrama de Contexto
C4Context
    title Diagrama de contexto — ATS MVP

    Person(recruiter, "Recruiter", "Gestiona vacantes y candidatos")
    Person(admin, "Admin", "Administra usuarios y configuracion")
    Person(candidate, "Candidate", "Postula a vacantes")

    System(ats, "ATS Platform", "Plataforma de seguimiento de candidatos")

    System_Ext(emailProvider, "Email Provider", "Servicio externo de envio de correos")

    Rel(recruiter, ats, "Gestiona vacantes y candidatos", "HTTPS")
    Rel(admin, ats, "Administra usuarios y configuracion", "HTTPS")
    Rel(candidate, ats, "Postula a vacantes", "HTTPS")
    Rel(ats, emailProvider, "Envia correos", "SMTP / API")

```

### Explicación

Este diagrama define el ATS como una **caja negra central**, mostrando únicamente actores externos. Se prioriza claridad funcional: Recruiters y Admins operan internamente, mientras el Candidate interactúa solo vía aplicación. El Email Provider es la única integración externa del MVP, lo cual reduce complejidad inicial. Esta separación permite evolucionar integraciones sin afectar el core del sistema.

---

# 2. Diagrama de Contenedores (C4 Nivel 2)

```mermaid
C4Container
    title Diagrama de contenedores — ATS MVP

    Person(recruiter, "Recruiter", "Usuario interno del ATS")
    Person(candidate, "Candidate", "Postulante externo")

    System_Boundary(ats, "ATS MVP") {
        Container(frontend, "Angular SPA", "TypeScript / Angular", "Interfaz de usuario del reclutador")
        Container(backend, "Node.js API", "Node.js / Express", "Logica de negocio y API REST")
        ContainerDb(db, "PostgreSQL", "Base de datos relacional", "Almacena vacantes, candidatos y pipeline")
    }

    System_Ext(email, "Email Service", "Proveedor SMTP externo")

    Rel(recruiter, frontend, "Usa", "HTTPS")
    Rel(candidate, frontend, "Postula", "HTTPS")
    Rel(frontend, backend, "Consume", "REST / JSON")
    Rel(backend, db, "Lee y escribe", "SQL")
    Rel(backend, email, "Envia correos", "SMTP / API")

```

### Explicación

Aquí se define una arquitectura clásica **3-tier desacoplada**. El frontend Angular es una SPA que consume la API REST, manteniendo lógica de negocio únicamente en backend. PostgreSQL actúa como fuente única de verdad, consistente con los aggregates definidos (Application, Candidate, JobOpening). El servicio de email está separado para permitir retries, colas futuras o cambio de proveedor sin afectar la API.

---

# 3. Diagrama de Componentes del Backend (C4 Nivel 3)

```mermaid

C4Component
    title Diagrama de componentes — Backend Node.js / Express

    Container_Boundary(backend, "Node.js API") {
        Component(api, "REST API Layer", "Express Router", "Enruta y valida todas las peticiones entrantes")
        Component(authModule, "AuthModule", "Node.js / JWT", "Autenticacion y autorizacion por rol")
        Component(jobModule, "JobOpeningModule", "Node.js", "Gestion de vacantes y pipeline snapshot")
        Component(candidateModule, "CandidateModule", "Node.js", "Gestion de perfiles de candidatos")
        Component(pipelineModule, "PipelineModule", "Node.js", "Movimiento de etapas y StageTransition")
        Component(evaluationModule, "EvaluationModule", "Node.js", "Registro de evaluaciones y feedback")
        Component(communicationModule, "CommunicationModule", "Node.js", "Envio de comunicaciones al candidato")
    }

    ContainerDb(db, "PostgreSQL", "Base de datos relacional", "Almacena todas las entidades del dominio")
    System_Ext(email, "Email Service", "Proveedor SMTP externo")

    Rel(api, authModule, "Valida token", "interno")
    Rel(api, jobModule, "Delega", "interno")
    Rel(api, candidateModule, "Delega", "interno")
    Rel(api, pipelineModule, "Delega", "interno")
    Rel(api, evaluationModule, "Delega", "interno")
    Rel(api, communicationModule, "Delega", "interno")

    Rel(jobModule, db, "Lee y escribe", "SQL")
    Rel(candidateModule, db, "Lee y escribe", "SQL")
    Rel(pipelineModule, db, "Lee y escribe", "SQL")
    Rel(evaluationModule, db, "Lee y escribe", "SQL")
    Rel(communicationModule, db, "Lee y escribe", "SQL")
    Rel(communicationModule, email, "Envia correos", "SMTP / API")

```

### Explicación

El backend sigue un enfoque **modular por dominio (DDD-lite)** alineado a los Aggregate Roots del modelo. Cada módulo encapsula lógica, validaciones y acceso a datos de su contexto. `PipelineModule` es crítico porque coordina `StageTransition` y consistencia con `Application`. `AuthModule` centraliza seguridad y multi-tenant enforcement. Esta estructura permite escalar a microservicios en el futuro si es necesario, sin cambiar el modelo mental actual.

---

# 4. Diagrama de Secuencia — Flujo Principal

```mermaid
%% Secuencia - Flujo Principal ATS
sequenceDiagram
    participant Recruiter
    participant Frontend
    participant Backend
    participant DB
    participant EmailService

    Recruiter->>Frontend: Create Job Opening
    Frontend->>Backend: POST /job-openings
    Backend->>DB: Insert JobOpening + PipelineStages

    Candidate->>Frontend: Apply to Job
    Frontend->>Backend: POST /applications
    Backend->>DB: Insert Application + StageTransition

    Recruiter->>Frontend: Move Candidate Stage
    Frontend->>Backend: PATCH /applications/{id}/stage
    Backend->>DB: Insert StageTransition + Update Application

    Backend->>EmailService: Send Email
```

### Explicación

Este flujo muestra el **happy path completo del ATS**. La clave arquitectónica es que `StageTransition` es la fuente de verdad, mientras `Application.currentStageId` es un cache consistente (actualizado en la misma transacción). El envío de email ocurre después de eventos clave (ej. cambio de etapa). Este diseño garantiza trazabilidad completa del pipeline sin sacrificar performance en lecturas.

---

# 5. Diagrama de Flujo — Ciclo de Vida de Application

```mermaid
%% Flujo - Lifecycle Application
flowchart LR
    Draft --> Open
    Open --> Active
    Active --> Hired
    Active --> Rejected
    Active --> Withdrawn
```

### Explicación

El ciclo de vida refleja claramente la naturaleza transaccional de `Application`. El estado `Active` representa participación en pipeline (con stages). Las salidas (`Hired`, `Rejected`, `Withdrawn`) están alineadas con `stageType` del pipeline, garantizando consistencia entre estado y etapa. Este modelo evita estados ambiguos y simplifica lógica de negocio en frontend y backend.

---

# Decisiones Arquitectónicas Clave (Resumen Ejecutivo)

### 1. Multi-tenant por `Company`

* Aislamiento lógico con `companyId` en todas las entidades
* Escalable sin necesidad de múltiples bases en MVP

### 2. Consistencia fuerte en pipeline

* `StageTransition` = source of truth
* `Application.currentStageId` = optimized read model

### 3. Modularización backend (DDD-aligned)

* Cada módulo corresponde a un dominio claro
* Facilita mantenimiento y evolución

### 4. Arquitectura simple pero extensible

* Monolito modular → preparado para microservicios
* Email desacoplado desde el inicio

### 5. Stack balanceado

* Angular → rapidez en UI empresarial
* Node.js → velocidad de desarrollo + I/O eficiente
* PostgreSQL → integridad + flexibilidad (JSONB futuro)
