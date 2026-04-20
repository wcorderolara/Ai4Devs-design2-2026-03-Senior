# Generacion de User Story Template
OK empecemos, ya tengo el PRD como ellos dicen pues ya tengo creado lo que son los casos de uso, el modelo de Datos, el Diseño del Sistema a Alto Nivel.

Lo que me gustaria poder generar y tener ahora es la posibilidad de poder crear un template de User Stories basado en alguna metodologia para poder luego solo hacer referencia a ese template, es como si tuviera dentro de mi proyecto un folder llamado ´templates´y dentro de los prompts o agentes solo les digo que tomen ese template para generar todo lo correspondiente a esa user storie.

Entonces, ahora vamos a generar el template de la User story, hazme todas las preguntas necesarias para poder generar dicho Template

### Respuesta de ChatGPT
Perfecto, esto lo estás planteando como alguien que ya piensa en **sistemas reutilizables + SDD + AI workflows**, no solo en “escribir historias”. Vamos bien.

Antes de darte un template genérico (que sería lo fácil pero mediocre), voy a hacerte **las preguntas correctas** para diseñar un template que realmente te sirva para:

* Generación automática con prompts
* Refinamiento en planning
* Uso real por devs (no solo académico)

---

# 🎯 Primero: Alinear el objetivo del template

Responde esto rápido:

### 1. ¿Para qué vas a usar principalmente este template?

Elige una:

A. Solo para cumplir la tarea académica
B. Para simular un backlog real tipo startup
C. Para usarlo como base en futuros proyectos reales

👉 Esto cambia mucho el nivel de profundidad

---

# 🧱 Metodología base (clave)

### 2. ¿Qué enfoque quieres usar para las User Stories?

Elige uno (o dime si quieres híbrido):

* **Scrum clásico (As a / I want / So that + AC)**
* **BDD (Given / When / Then fuerte)**
* **Dual (Historia + criterios BDD) ← 🔥 recomendado**
* **Job To Be Done (When / I want / So I can)**

👉 Para ATS + SDD, yo recomiendo:
**Dual = Story + Acceptance Criteria en formato BDD**

---

# 🧩 Nivel de profundidad del template

### 3. ¿Qué tan detallado quieres el template?

Elige:

* 🟢 Básico (rápido, tipo Jira simple)
* 🟡 Medio (listo para backlog refinement)
* 🔴 Avanzado (listo para sprint planning + dev ready)

👉 Tú perfil sugiere 🔴, pero confírmame

---

# 🧠 Contexto del sistema (importante para IA)

### 4. ¿Quieres que cada User Story incluya contexto del dominio?

Ejemplo:

* Entidades involucradas (Application, Candidate, Job)
* Reglas de negocio

Opciones:

* Sí, obligatorio
* Opcional
* No

---

# ⚙️ Parte técnica dentro del template

### 5. ¿Quieres que el template incluya hints técnicos?

Ejemplo:

* API endpoints sugeridos
* Cambios en DB
* Validaciones

Opciones:

* Sí (nivel arquitecto)
* Solo notas técnicas ligeras
* No (separar completamente negocio y tech)

---

# 📊 Priorización integrada

### 6. ¿Quieres que cada User Story ya incluya prioridad?

Opciones:

* Sí (ej: RICE / WSJF)
* Solo un campo simple (High / Medium / Low)
* No (priorización va aparte en backlog)

---

# 🧪 Testing / QA

### 7. ¿Qué nivel de testing quieres en el template?

Opciones:

* Solo Acceptance Criteria
* * Casos de prueba explícitos
* * Edge cases + error handling

---

# 🤖 Pensando en prompts / agentes (esto es clave para tu idea)

### 8. ¿Cómo quieres usar el template con IA?

Esto es MUY importante para diseñarlo bien.

Ejemplo:

* “Given this PRD, generate a User Story using TEMPLATE_X”
* “Fill this template based on this use case”

👉 Dime:

* ¿Quieres que el template sea:

  * 🧾 Texto plano (copy/paste)
  * 🧩 JSON/YAML (más usable por agentes)
  * 🔥 Híbrido (markdown estructurado para humanos + AI)

---

# 🧬 Modularidad (nivel pro que estás buscando)

### 9. ¿Quieres que el template sea:

* Único (una sola estructura)
* Modular (bloques opcionales como Lego)

Ejemplo modular:

* Core Story
* Business Rules
* Tech Notes
* Metrics

👉 Esto es muy poderoso para SDD

---

# 📈 Métricas / impacto

### 10. ¿Quieres incluir impacto de negocio?

Ejemplo:

* KPI afectado
* Métrica objetivo

Opciones:

* Sí (nivel producto real)
* No

---

# 🎯 Mi hipótesis (confírmame si voy bien)

Por lo que veo de tu perfil, creo que quieres algo así:

* Nivel 🔴 avanzado
* Dual: Story + BDD
* Modular
* Reutilizable en prompts
* Con contexto de dominio
* Con hints técnicos (ligeros)
* En formato markdown estructurado

👉 Pero no quiero asumir — confírmame.

---

# 🚀 Qué pasará después de tus respuestas

Con eso te voy a construir:

### 1. Template PRO de User Story (no básico)

* Listo para ATS
* Compatible con SDD

### 2. Versión optimizada para prompts

* Copy-paste usable
* Con instrucciones embebidas

### 3. Ejemplo aplicado a tu ATS

### 4. Bonus (si quieres):

* Cómo referenciarlo desde un “agents/prompt system”

---

Respóndeme esas 10 preguntas (puedes hacerlo tipo lista rápida)
y te construyo algo que no vas a encontrar en ningún curso básico.


### Respuesta para continuar con la generacion
1. Lo voy a usar para los 3, no solo para la resolucion, si no tambien para hacer un backlog y para proyectos futuros para ir trabajando en SDD

2. la metodologia Dual me parece más sólida y concreta

3. prefiero el Avanzado, para no dejar nada al aire para que la IA no infiera

4. Yo al menos considero importante que una US constenga el contexto completo del dominio, esto para dejar claro tanto al DEV como al Agente de IA que se va a usar, como se va a usar y las reglas del negocio basado en (me corriges si estoy mal) en los casos de uso definidos previamente

5. Las Validaciones considero que van dentro de las reglas del negocio, por el momento me gustaria separar la parte tech del negocio, considero que para las US no es tan importante, pero me corrgies si estoy mal.

6. Si, incluyamos prioridad

7. Hagamos Acceptance Criteria incluyendo Edge Cases y haciendo mencion de como manejar dichos escenarios

8. Creo que el Híbrido es el más usado, pero igual me corriges.

9. que me aconsejas para esto?

10. Si, incluyamos Métricas/Impacto

El resultado del Template obtenido se guarda en `templates/user-story-template.md`


# PROMPT: Generar User Stories

You are a Senior Product Manager and Business Analyst with strong experience in SaaS B2B platforms and Spec-Driven Development (SDD).

## 🎯 Objective
Generate a structured set of User Stories based on the provided ATS (Applicant Tracking System) product documentation.

## 📥 Input Context
You will receive a full product document that includes:
- Problem definition
- Feature list
- Use Cases
- Data Model (entities and relationships)
- High-Level System Design

This is NOT a generic product. You MUST use this context as the single source of truth.

---

## 🧠 Instructions

### 1. Feature Grouping
- Identify logical product features from the document (e.g. Candidate Management, Job Management, Pipeline Management, etc.)
- Group User Stories under these features
- Do NOT create isolated or random stories

---

### 2. User Story Generation
For each relevant feature:

- Generate multiple User Stories (not just one)
- Each User Story MUST map to one or more Use Cases from the document
- Do NOT invent functionality outside the defined scope

---

### 3. Strict Template Usage
Each User Story MUST follow this template EXACTLY:

# 🧾 User Story: <TITLE>

## 🆔 Metadata
- ID: US-<incremental>
- Feature: <Feature Name>
- Priority: <High | Medium | Low>
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** <type of user>  
**I want** <goal/action>  
**So that** <business value>  

## 🧠 Business Context
- Related Use Case(s): <Use Case names>
- Entities involved: <from data model only>
- Business Rules:
  - <rule aligned with document>
- Assumptions:
  - <if needed>
- Dependencies:
  - <if applicable>

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** <context>  
  **When** <action>  
  **Then** <expected result>  

### ⚠️ Edge Cases

#### Case 1: <name>
- **Given** <context>  
  **When** <action>  
  **Then** <result>  
- Handling:
  - <system behavior>

#### Case 2: <name>
- **Given** <context>  
  **When** <action>  
  **Then** <result>  
- Handling:
  - <system behavior>

### 🚫 Validation Rules
- <validation aligned with business rules>

## 📊 Business Impact
- KPI Affected: <metric from document>
- Expected Impact: <qualitative impact>
- Success Criteria:
  - <metric + target>

---

### 4. Domain Enforcement (CRITICAL)
- Only use entities defined in the Data Model (Candidate, Application, JobOpening, etc.)
- Do NOT invent new entities
- Respect relationships (e.g., Application belongs to Candidate + JobOpening)

---

### 5. Prioritization Logic
- Assign priority based on MVP scope described in the document
- Core features = High
- Secondary features = Medium/Low

---

### 6. Output Format

- Group User Stories by Feature
- Use clean Markdown
- Do NOT add explanations outside the stories
- Do NOT include commentary

---

## ⚠️ Constraints
- No generic or vague stories
- No missing sections
- No invented features
- No skipping edge cases

---

## 📁 Output File Structure

For each User Story:

- Generate it as an individual Markdown file
- Use the following path:

/user-stories/US-<ID>-<slug-title>.md

### Naming Rules:
- Replace spaces with hyphens
- Use lowercase
- Keep it short but descriptive

### Example:
US-001-submit-application.md

---

### File Content Format:

Start each file with:

PATH: /user-stories/US-XXX-<slug>.md

Then include the full User Story using the defined template.

---

### Important:
- Do NOT merge multiple User Stories in one file
- Each story must be isolated
- Ensure IDs are sequential

---
## Ejecucion

`Generate the user Stories using @LTI-WACL/prompts/generate-user-stories.md as Input use the file @LTI-WACL/LTI-WACL.md`



# PROMPT: Generar Backlog
You are a Senior Product Manager with experience in SaaS B2B platforms and Agile product development.

## 🎯 Objective
Create a structured and prioritized Product Backlog based on the provided User Stories and ATS product documentation.

---

## 📥 Input Context
You will receive:
1. A set of User Stories
2. A PRD-like document that includes:
   - Features and priorities (MVP vs secondary)
   - Use Cases
   - Business goals (e.g. time-to-hire, conversion rate)

You MUST use this context as the source of truth.

---

## 🧠 Instructions

### 1. Feature Grouping
- Group User Stories by Feature
- Do NOT leave stories ungrouped
- Use logical product modules (e.g. Job Management, Candidate Management, Pipeline Management)

---

### 2. Prioritization Method (MANDATORY)

Use one of the following:
- MoSCoW (Must / Should / Could / Won’t)
OR
- Business Value vs Effort (High/Medium/Low)

👉 Default: Use **MoSCoW**

---

### 3. Prioritization Logic

You MUST consider:

- MVP scope (core features = Must)
- Business impact (time-to-hire, hiring quality)
- Dependency order (what must exist first)
- User journey (what unlocks value first)

---

### 4. Backlog Structure

For each User Story include:

- ID
- Title
- Feature
- Priority (MoSCoW)
- Short Description (1 line)
- Business Value (why it matters)
- Dependencies (if any)

---

### 5. Output Format

Return a structured Markdown table grouped by Feature.

Example:

## Feature: Candidate Management

| ID | Title | Priority | Description | Business Value | Dependencies |
|----|------|---------|------------|---------------|-------------|

---

### 6. Constraints

- Do NOT invent new User Stories
- Do NOT skip any provided stories
- Keep descriptions concise
- Prioritization must be justified logically

---

## 🧠 Extra Requirement (IMPORTANT FOR TASK)

After the table, include:

### 📊 Prioritization Rationale

Explain briefly:
- Why certain stories are MUST
- Why others are lower priority
- Any dependency reasoning

---

## 📁 Output File Structure

Generate the backlog as a single Markdown file.

Use the following path:

/backlog/product-backlog.md

---

### File Content Format

Start the file with:

PATH: /backlog/product-backlog.md

---

### Structure Inside the File

The file must contain:

1. Title: Product Backlog
2. Grouped sections by Feature
3. Markdown tables per Feature
4. A final section:
   ## 📊 Prioritization Rationale

---

### Naming Convention Rules

- Use lowercase
- Use hyphen-separated naming
- Keep it standard and reusable across projects

---

### Important Constraints

- Do NOT split the backlog into multiple files
- Keep everything in a single structured document
- Ensure readability and clean formatting

---
## Ejecucion del PROMPT
`Generate the backlog using @LTI-WACL/prompts/generate-backlog.md and as inputs use @LTI-WACL/LTI-WACL.md  and all the files within user-stories folder`

---

# PROMPT: Generar Tasks
You are a Senior Software Architect and Technical Lead working with Spec-Driven Development (SDD).

## 🎯 Objective
Break down a given User Story into detailed, implementation-ready technical tasks.

---

## 📥 Input Context

You will receive:

1. A User Story (primary input)
2. Supporting documents (if available):
   - PRD
   - Data Model
   - Technical Specs

You MUST use these as the source of truth.

---

## 🧠 Instructions

### 1. Task Decomposition

Break the User Story into atomic, dev-ready tasks grouped by:

- API Layer
- Domain / Business Logic
- Persistence (Database)
- Frontend (UI)
- Validation & Rules
- Testing

Each task must be:
- Independently executable
- Clearly scoped
- Technically precise

---

### 2. Technical Grounding (CRITICAL)

- Use ONLY entities defined in the Data Model
- Respect relationships and constraints
- Do NOT invent architecture or entities
- Align with SDD principles

---

### 3. Task Structure

Each task MUST include:

- ID: TASK-<incremental>
- Title
- Description (clear and technical)
- Steps (implementation steps)
- Acceptance Criteria (BDD format)
- Dependencies (if any)
- Estimation (Story Points using Fibonacci)

---

### 4. Dependency Handling

- Clearly define simple dependencies between tasks
- Example:
  - DB task → required before API
  - API → required before FE

Keep it simple and readable

---

### 5. Estimation Rules

Use Fibonacci scale:
1, 2, 3, 5, 8

Base estimation on:
- Complexity
- Unknowns
- Integration points

---

### 6. Output File Structure

Generate tasks as individual files using the following structure:

/tasks/us-<ID>-<slug>/

Each task must be a separate file:

/tasks/us-<ID>-<slug>/task-<XXX>-<slug>.md

---

### Naming Rules

- Use lowercase
- Use hyphen-separated names
- Keep names short but descriptive

---

### Example

/tasks/us-003-move-candidate-stage/
  task-001-update-application-stage.md
  task-002-create-stage-transition.md

---

### File Content Format

Each file must start with:

PATH: /tasks/us-XXX-<slug>/task-XXX-<slug>.md

Then include:

# 🔧 Task: <TITLE>

## 🆔 Metadata
- ID: TASK-XXX
- Related User Story: US-XXX
- Layer: <API | Domain | Persistence | Frontend | Validation | Testing>
- Estimation: <Story Points>

---

## 📝 Description
<technical description>

---

## ⚙️ Steps
1. <step>
2. <step>

---

## ✅ Acceptance Criteria

- **Given** <context>  
  **When** <action>  
  **Then** <expected result>

---

## 🔗 Dependencies
- TASK-XXX (if applicable)

---

### Constraints

- Do NOT merge tasks into one file
- Do NOT skip any layer if required
- Do NOT produce vague or generic tasks
- Do NOT include explanations outside files

---

## 🚀 Expected Outcome

A complete, structured, dev-ready breakdown of the User Story into executable tasks.

---

## Ejecucion del PROMPT
`Use the @LTI-WACL/prompts/generate-tasks.md file to create the tasks with this Inputs: User Story located on @LTI-WACL/user-stories/US-001-create-job-opening.md the PRD located on @LTI-WACL/docs/LTI-WACL.md and the specifications located on @LTI-WACL/docs/specifications.md`