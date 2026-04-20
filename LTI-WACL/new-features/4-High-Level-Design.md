# 1. Diagrama de Contexto (C4 Nivel 1)

```mermaid
C4Context
    title ATS Platform - Contexto actualizado

    Person(recruiter, "Recruiter")
    Person(candidate, "Candidate")
    Person(admin, "Admin")
    Person(manager, "Hiring Manager")

    System(ats, "ATS Platform")

    System_Ext(email, "Email Service")
    System_Ext(llm, "LLM Provider")
    System_Ext(automation, "Automation Engine (external triggers)")
    System_Ext(realtime, "Realtime Notification Service")

    Rel(recruiter, ats, "Gestiona candidatos")
    Rel(manager, ats, "Colabora y evalúa")
    Rel(candidate, ats, "Aplica a vacantes")
    Rel(admin, ats, "Administra sistema")

    Rel(ats, email, "Envía emails")
    Rel(ats, llm, "Solicita IA")
    Rel(ats, automation, "Eventos/Triggers")
    Rel(ats, realtime, "Push eventos")
```

### Decisiones clave

* **LLM como sistema externo vs integrado** → Se desacopla como proveedor externo
  → Permite cambiar proveedor sin afectar core
  → Impacto: introduce capa gateway interna

* **Realtime como servicio separado vs polling** → Se usa push-based
  → Mejora UX y eficiencia
  → Impacto: nuevo contenedor obligatorio

---

# 2. Diagrama de Contenedores (C4 Nivel 2)

```mermaid
C4Container
    title ATS Platform - Contenedores extendidos

    Person(recruiter)
    
    Container(spa, "Angular SPA", "Frontend")
    Container(api, "Node.js API", "Backend REST")

    ContainerDb(db, "PostgreSQL", "Database")
    Container(email, "Email Service", "External")

    Container(ws, "WebSocket/SSE Server", "Realtime")
    Container(auto, "Automation Engine", "Rules & Triggers")
    Container(ai, "AI Gateway Service", "LLM abstraction")
    Container(metrics, "Metrics Service", "Analytics")

    System_Ext(llm, "LLM Provider")

    Rel(recruiter, spa, "Uses")
    Rel(spa, api, "REST")

    Rel(api, db, "SQL")
    Rel(api, email, "SMTP/API")

    Rel(api, ws, "Publish events")
    Rel(ws, spa, "Push updates")

    Rel(api, auto, "Emit domain events")
    Rel(auto, api, "Trigger actions")

    Rel(api, ai, "AI requests")
    Rel(ai, llm, "LLM API")

    Rel(api, metrics, "Send events")
    Rel(metrics, db, "Store aggregates")
```

### Decisiones clave

* **WebSocket separado vs embebido** → Separado
  → Escalabilidad independiente
  → Impacto: nuevo deployment unit

* **Automation Engine desacoplado** → Event-driven
  → Evita acoplamiento con PipelineModule
  → Impacto: comunicación por eventos

* **AI Gateway vs llamada directa** → Gateway
  → Abstracción total del proveedor
  → Impacto: nuevo servicio intermedio

---

# 3. Diagrama de Componentes (C4 Nivel 3 - Backend)

```mermaid
C4Component
    title Node.js API - Nuevos módulos

    Container(api, "Node.js API")

    Component(auth, "AuthModule")
    Component(candidate, "CandidateModule")
    Component(pipeline, "PipelineModule")
    Component(eval, "EvaluationModule")

    Component(collab, "CollaborationModule")
    Component(auto, "AutomationModule")
    Component(ai, "AIAssistantModule")
    Component(metrics, "MetricsModule")

    ContainerDb(db, "PostgreSQL")
    Container(ws, "WebSocket Server")
    Container(ai_srv, "AI Gateway")
    Container(auto_srv, "Automation Engine")

    Rel(collab, ws, "Emit events")
    Rel(collab, db, "Store comments")

    Rel(auto, auto_srv, "Send triggers")
    Rel(auto_srv, pipeline, "Invoke actions")

    Rel(ai, ai_srv, "Request AI")
    Rel(ai, db, "Store suggestions")

    Rel(metrics, db, "Read/Write metrics")

    Rel(pipeline, auto, "Emit events")
    Rel(candidate, ai, "Provide data context")
```

### Decisiones clave

* **AutomationModule vs lógica en Pipeline** → Separado
  → Mantiene single responsibility
  → Impacto: eventos de dominio obligatorios

* **AIAssistantModule como gateway interno** → No acoplado a provider
  → Permite multi-provider
  → Impacto: nuevo modelo de datos (AI suggestions)

---

# 4. Secuencia — Automatización

```mermaid
sequenceDiagram
    participant Candidate
    participant API
    participant AutomationModule
    participant AutomationEngine
    participant EmailService

    Candidate->>API: Avanza etapa
    API->>AutomationModule: Emit event
    AutomationModule->>AutomationEngine: Evaluate rules
    AutomationEngine-->>AutomationModule: Trigger action
    AutomationModule->>EmailService: Send email
    AutomationModule->>API: Update pipeline
```

### Decisiones clave

* **Evaluación fuera del core**
  → Permite reglas dinámicas
  → Impacto: motor configurable

---

# 5. Secuencia — Asistencia IA

```mermaid
sequenceDiagram
    participant Recruiter
    participant API
    participant AIAssistant
    participant AIGateway
    participant LLM

    Recruiter->>API: Request suggestion
    API->>AIAssistant: Build context
    AIAssistant->>AIGateway: Send prompt
    AIGateway->>LLM: API call
    LLM-->>AIGateway: Response
    AIGateway-->>AIAssistant: Processed result
    AIAssistant->>API: Store suggestion
    API-->>Recruiter: Return suggestion
```

### Decisiones clave

* **Context building en backend vs frontend**
  → Seguridad + consistencia
  → Impacto: uso intensivo de CandidateModule

---

# 6. Secuencia — Colaboración en tiempo real

```mermaid
sequenceDiagram
    participant Manager
    participant API
    participant WS
    participant Recruiter
    participant DB

    Manager->>API: Add comment
    API->>DB: Save comment
    API->>WS: Emit event
    WS-->>Recruiter: Push notification
```

### Decisiones clave

* **Persistencia antes del evento**
  → Garantiza consistencia
  → Impacto: orden estricto en flujo

---

# 7. Flujo — Ciclo de vida de Automatización

```mermaid
flowchart TD
    A[Configurada] --> B[Activa]
    B --> C[Trigger detectado]
    C --> D[Condiciones evaluadas]
    D --> E[Acción ejecutada]
    E --> F{Resultado}
    F -->|Éxito| G[Historial registrado]
    F -->|Falla| H[Error registrado]
```

### Decisiones clave

* **Estado explícito vs implícito**
  → Facilita debugging
  → Impacto: tabla de AutomationHistory

---

# 🔎 Conclusión directa para el equipo

Respondiendo las preguntas clave:

### 1. ¿Qué contenedor nuevo necesitas para realtime?

👉 **WebSocket/SSE Server**

---

### 2. ¿Cómo se comunica automatización sin acoplarse?

👉 Mediante **eventos de dominio desde PipelineModule → AutomationModule → Automation Engine**

---

### 3. ¿Dónde vive la lógica de IA?

👉 En **AIAssistantModule + AI Gateway (abstracción total del provider)**

---

### 4. Flujo completo de IA

👉 Recruiter → API → AIAssistant → AI Gateway → LLM → respuesta → almacenamiento → UI

---

Si quieres, en el siguiente paso podemos bajar esto a:

* contratos API (OpenAPI)
* eventos de dominio exactos
* diseño de tablas nuevas (AI suggestions, automation logs, metrics)

o incluso convertirlo en **tasks listas para sprint (Jira-ready)**.
