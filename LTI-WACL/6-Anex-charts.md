# Lean Canvas (mermaid)
flowchart TB
    subgraph L1[" "]
        A["Problemas<br/>• Procesos manuales<br/>• Pérdida de candidatos<br/>• Falta de visibilidad<br/>• Mala colaboración"]
        B["Solución<br/>• Gestión de vacantes<br/>• Formulario de aplicación<br/>• Perfil de candidato<br/>• Pipeline<br/>• Evaluación y feedback<br/>• Comunicación básica"]
        C["Propuesta de Valor Única<br/>ATS simple para centralizar reclutamiento, ordenar el pipeline y acelerar contrataciones"]
        D["Ventaja Competitiva<br/>• Fácil adopción<br/>• Flujo claro<br/>• Base mínima pero funcional"]
        E["Segmentos de Clientes<br/>• Startups en crecimiento<br/>• PyMEs<br/>• Empresas con contratación frecuente"]
    end

    subgraph L2[" "]
        F["Métricas Clave<br/>• Time to hire<br/>• Candidatos por vacante<br/>• Conversión por etapa<br/>• Tiempo en etapa"]
        G["Canales<br/>• Venta directa B2B<br/>• Sitio web<br/>• Demo comercial<br/>• Referidos"]
    end

    subgraph L3[" "]
        H["Estructura de Costos<br/>• Desarrollo<br/>• Infraestructura cloud<br/>• Soporte<br/>• Mantenimiento"]
        I["Fuentes de Ingreso<br/>• Suscripción mensual<br/>• Plan por usuarios<br/>• Plan por vacantes"]
    end

    A --> B
    B --> C
    D --> C
    E --> C
    C --> F
    C --> G
    B --> H
    E --> I

# CJM (mermaid)
journey
    title Customer Journey Map - Implementación y uso inicial de un ATS
    section Descubrimiento
      Identifica problemas en reclutamiento: 4: Empresa
      Busca opciones de ATS en el mercado: 3: Empresa
      Compara beneficios y casos de uso: 3: Empresa
    section Evaluación
      Solicita demo o prueba: 4: Empresa, Ventas
      Evalúa funcionalidades clave: 3: Empresa
      Valida costo, facilidad de uso y adopción: 2: Empresa
      Alinea decisión con stakeholders internos: 2: Empresa, Hiring Manager, Finanzas
    section Compra
      Selecciona proveedor ATS: 4: Empresa
      Define plan y licencias: 3: Empresa, Ventas
      Confirma contratación del servicio: 5: Empresa, Ventas
    section Onboarding
      Crea cuenta y configura empresa: 4: Empresa, Customer Success
      Define usuarios y roles: 3: Empresa
      Configura pipeline de reclutamiento: 3: Empresa
      Crea primeras vacantes: 4: Empresa
    section Adopción Inicial
      Publica vacantes activas: 5: Empresa
      Recibe primeras aplicaciones: 5: Empresa, Candidato
      Revisa perfiles y CVs: 4: Empresa
      Mueve candidatos por etapas: 4: Empresa
    section Colaboración
      Coordina entrevistas con el equipo: 3: Empresa, Hiring Manager
      Registra feedback y evaluaciones: 4: Hiring Manager, Entrevistador
      Consolida notas internas y decisiones: 4: Empresa
    section Decisión
      Compara candidatos finalistas: 4: Empresa, Hiring Manager
      Selecciona candidato ideal: 5: Empresa
      Envía oferta o rechazo: 4: Empresa, Candidato
    section Optimización
      Revisa métricas del proceso: 3: Empresa
      Detecta cuellos de botella: 2: Empresa
      Ajusta etapas y flujo de trabajo: 4: Empresa
      Reutiliza base de candidatos para nuevas vacantes: 5: Empresa

# DIAGRAMA ENTIDAD RELACION 

### CHATGPT
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
        datetime createdAt
    }

    %% =========================
    %% Job Openings and Pipeline
    %% =========================
    JobOpening {
        uuid id PK
        uuid companyId FK
        uuid pipelineTemplateId FK
        uuid createdByUserId FK
        string title
        string department
        string location
        string workMode
        text description
        string status
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
        uuid pipelineTemplateId FK
        string name
        int sequence
        string stageType
    }

    Application {
        uuid id PK
        uuid companyId FK
        uuid candidateId FK
        uuid jobOpeningId FK
        uuid currentStageId FK
        string status
        datetime appliedAt
        string source
        string rejectionReason
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

    %% =========================
    %% Evaluation and Collaboration
    %% =========================
    Evaluation {
        uuid id PK
        uuid applicationId FK
        uuid evaluatorUserId FK
        uuid stageId FK
        decimal score
        string recommendation
        text summary
        datetime createdAt
    }

    Note {
        uuid id PK
        uuid applicationId FK
        uuid candidateId FK
        uuid authorUserId FK
        text content
        datetime createdAt
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
        datetime sentAt
        datetime createdAt
    }

    %% =========================
    %% Relationships
    %% =========================
    Company ||--o{ User : "tener"
    Company ||--o{ Candidate : "tener"
    Company ||--o{ JobOpening : "tener"
    Company ||--o{ PipelineTemplate : "tener"
    Company ||--o{ Application : "contener"

    User ||--o{ JobOpening : "crear"
    User ||--o{ StageTransition : "ejecutar"
    User ||--o{ Evaluation : "emitir"
    User ||--o{ Note : "redactar"
    User ||--o{ Communication : "enviar"

    Candidate ||--o{ Application : "postular"
    Candidate ||--o{ Note : "recibir"
    Candidate ||--o{ Communication : "recibir"

    PipelineTemplate ||--o{ PipelineStage : "definir"
    PipelineTemplate ||--o{ JobOpening : "estructurar"

    JobOpening ||--o{ Application : "recibir"

    PipelineStage ||--o{ Application : "ubicar"
    PipelineStage ||--o{ StageTransition : "originar"
    PipelineStage ||--o{ StageTransition : "destinar"
    PipelineStage ||--o{ Evaluation : "contextualizar"

    Application ||--o{ StageTransition : "registrar"
    Application ||--o{ Evaluation : "acumular"
    Application ||--o{ Note : "acumular"
    Application ||--o{ Communication : "registrar"

### CLAUDE Sonnet 4.6
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
        datetime createdAt
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

    %% =========================
    %% Evaluation and Collaboration
    %% =========================
    Evaluation {
        uuid id PK
        uuid applicationId FK
        uuid evaluatorUserId FK
        uuid stageId FK
        decimal score
        string recommendation
        text summary
        datetime createdAt
        datetime completedAt
    }

    Note {
        uuid id PK
        string contextType
        uuid contextId FK
        uuid authorUserId FK
        text content
        datetime createdAt
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
    %% Relationships
    %% =========================
    Company ||--o{ User : crear
    Company ||--o{ Candidate : pertenecer
    Company ||--o{ JobOpening : contener
    Company ||--o{ PipelineTemplate : definir
    Company ||--o{ Application : contener

    User ||--o{ JobOpening : crear
    User ||--o{ StageTransition : ejecutar
    User ||--o{ Evaluation : emitir
    User ||--o{ Note : redactar
    User ||--o{ Communication : enviar

    Candidate ||--o{ Application : postular
    Candidate ||--o{ Communication : recibir

    PipelineTemplate ||--o{ PipelineStage : definir

    JobOpening ||--o{ Application : recibir
    JobOpening ||--o{ PipelineStage : copiar

    PipelineStage ||--o{ Application : ubicar
    PipelineStage o|--o{ StageTransition : originar
    PipelineStage ||--o{ StageTransition : destinar
    PipelineStage o|--o{ Evaluation : contextualizar

    Application ||--o{ StageTransition : registrar
    Application ||--o{ Evaluation : acumular
    Application ||--o{ Communication : registrar

