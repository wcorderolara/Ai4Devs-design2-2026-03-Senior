Antes de entrar al brainstorming completo, te hago una **micro-validación rápida (crítica para que esto sea realmente útil para estimación)**:

👉 ¿El modelo de datos actual ya incluye entidades tipo `Application`, `Candidate`, `Job`, `Interview`, `User`, `Note`, `Attachment`?
👉 ¿Ya existe algo de tracking de estados tipo pipeline (ej: `stage`, `status`)?

Voy a asumir que **sí existe un modelo ATS estándar MVP** con esas entidades base (como Greenhouse/Lever), y diseñar encima de eso.

---

# 🧠 Brainstorming Funcionalidades ATS MVP

---

# 1. Eficiencia para departamentos de HR

## ### 1. Definición del problema

* **Pain point:** Los recruiters pierden tiempo en tareas repetitivas (filtrado manual, seguimiento, coordinación).
* **Actor afectado:** Recruiter
* **Impacto:**

  * Time-to-hire alto
  * Pérdida de candidatos buenos
  * Burnout operativo en HR

---

## ### 2. How Might We

* ¿Cómo podríamos reducir el tiempo que un recruiter invierte en revisar candidatos?
* ¿Cómo podríamos centralizar toda la información relevante sin cambiar de contexto?
* ¿Cómo podríamos priorizar automáticamente candidatos relevantes?
* ¿Cómo podríamos reducir tareas manuales sin perder control?

---

## ### 3. Ideas generadas (divergencia)

| Tipo       | Idea                                        |
| ---------- | ------------------------------------------- |
| Quick Win  | Filtros avanzados guardables por recruiter  |
| Quick Win  | Vista consolidada tipo "Candidate Snapshot" |
| Mediano    | Ranking automático de candidatos por score  |
| Mediano    | Pipeline view optimizado con drag & drop    |
| Mediano    | Bulk actions (mover, rechazar, etiquetar)   |
| Innovación | Inbox inteligente de candidatos priorizados |
| Innovación | Auto-highlighting de info clave en CV       |

---

## ### 4. Ideas priorizadas (convergencia)

| Nombre                  | Descripción                                  | Actor     |
| ----------------------- | -------------------------------------------- | --------- |
| Candidate Smart Ranking | Ranking automático basado en match con job   | Recruiter |
| Bulk Pipeline Actions   | Acciones masivas sobre candidatos            | Recruiter |
| Candidate Snapshot View | Vista unificada con info clave del candidato | Recruiter |

---

## ### 5. Casos de uso candidatos

* "Rankear candidatos automáticamente" → Recruiter → Módulo: Applications
* "Ejecutar acciones masivas en pipeline" → Recruiter → Módulo: Applications
* "Visualizar resumen de candidato" → Recruiter → Módulo: Candidate/Profile

---

## ### 6. Impacto en el modelo de datos

| Entidad     | Cambio                            | Tipo          |
| ----------- | --------------------------------- | ------------- |
| Application | campo `score`, `ranking_position` | Cambio menor  |
| Candidate   | campo `parsed_summary`            | Cambio menor  |
| Application | campo `tags[]`                    | Cambio menor  |
| —           | entidad `SavedFilter`             | Entidad nueva |

---

# 2. Colaboración en tiempo real

## ### 1. Definición del problema

* **Pain point:** Feedback disperso (Slack, emails, reuniones)
* **Actor:** Hiring Manager, Recruiter
* **Impacto:**

  * Decisiones lentas
  * Falta de trazabilidad
  * Sesgo en decisiones

---

## ### 2. How Might We

* ¿Cómo podríamos centralizar el feedback en un solo lugar?
* ¿Cómo podríamos hacer visible el estado de decisión en tiempo real?
* ¿Cómo podríamos evitar pérdida de contexto en evaluaciones?
* ¿Cómo podríamos facilitar consenso entre stakeholders?

---

## ### 3. Ideas generadas

| Tipo       | Idea                                   |
| ---------- | -------------------------------------- |
| Quick Win  | Comentarios en candidato               |
| Quick Win  | @mentions en notas                     |
| Mediano    | Scorecards estructurados               |
| Mediano    | Timeline de actividad del candidato    |
| Mediano    | Notificaciones internas                |
| Innovación | Consensus indicator (barra de acuerdo) |
| Innovación | Decision heatmap por candidato         |

---

## ### 4. Ideas priorizadas

| Nombre                        | Descripción                              | Actor               |
| ----------------------------- | ---------------------------------------- | ------------------- |
| Candidate Comments & Mentions | Comentarios con tagging interno          | Recruiter / Manager |
| Structured Scorecards         | Evaluación estandarizada post entrevista | Interviewer         |
| Candidate Activity Timeline   | Historial completo de interacción        | Todos               |

---

## ### 5. Casos de uso candidatos

* "Comentar en candidato" → Recruiter → Módulo: Candidate
* "Evaluar candidato con scorecard" → Interviewer → Módulo: Interview
* "Consultar historial del candidato" → HiringManager → Módulo: Candidate

---

## ### 6. Impacto en el modelo de datos

| Entidad   | Cambio                 | Tipo          |
| --------- | ---------------------- | ------------- |
| —         | entidad `Comment`      | Entidad nueva |
| Interview | campo `scorecard_json` | Cambio menor  |
| —         | entidad `Mention`      | Entidad nueva |
| —         | entidad `ActivityLog`  | Entidad nueva |

---

# 3. Automatizaciones

## ### 1. Definición del problema

* **Pain point:** Procesos manuales repetitivos (seguimientos, cambios de estado)
* **Actor:** Recruiter / Admin
* **Impacto:**

  * Retrasos
  * Errores humanos
  * Falta de consistencia

---

## ### 2. How Might We

* ¿Cómo podríamos automatizar flujos repetitivos?
* ¿Cómo podríamos reaccionar automáticamente a eventos?
* ¿Cómo podríamos reducir tareas operativas sin perder control?

---

## ### 3. Ideas generadas

| Tipo       | Idea                            |
| ---------- | ------------------------------- |
| Quick Win  | Auto-move por stage             |
| Quick Win  | Recordatorios automáticos       |
| Mediano    | Reglas tipo "if-this-then-that" |
| Mediano    | Auto-rechazo por criterios      |
| Mediano    | SLA tracking por etapa          |
| Innovación | Workflow builder visual         |
| Innovación | Automatización predictiva       |

---

## ### 4. Ideas priorizadas

| Nombre                        | Descripción                    | Actor     |
| ----------------------------- | ------------------------------ | --------- |
| Rule-Based Automation Engine  | Reglas automáticas por eventos | Admin     |
| Stage SLA Tracker             | Tracking de tiempo por etapa   | Recruiter |
| Auto Candidate Status Updates | Cambio automático de estado    | Recruiter |

---

## ### 5. Casos de uso candidatos

* "Configurar reglas de automatización" → Admin → Módulo: Settings
* "Actualizar estado automáticamente" → System → Módulo: Applications
* "Monitorear SLA de pipeline" → Recruiter → Módulo: Dashboard

---

## ### 6. Impacto en el modelo de datos

| Entidad     | Cambio                           | Tipo          |
| ----------- | -------------------------------- | ------------- |
| —           | entidad `AutomationRule`         | Entidad nueva |
| Application | campo `last_stage_change_at`     | Cambio menor  |
| —           | entidad `AutomationExecutionLog` | Entidad nueva |

---

# 4. Asistencia de IA

## ### 1. Definición del problema

* **Pain point:** Decisiones lentas + baja calidad en screening
* **Actor:** Recruiter
* **Impacto:**

  * Mala selección
  * Alto tiempo de análisis
  * Inconsistencia

---

## ### 2. How Might We

* ¿Cómo podríamos asistir al recruiter sin reemplazarlo?
* ¿Cómo podríamos mejorar la calidad del screening?
* ¿Cómo podríamos acelerar la toma de decisiones?

---

## ### 3. Ideas generadas

| Tipo       | Idea                                  |
| ---------- | ------------------------------------- |
| Quick Win  | Resumen automático de CV              |
| Quick Win  | Generación de feedback                |
| Mediano    | Matching candidato-job                |
| Mediano    | Sugerencia de preguntas de entrevista |
| Mediano    | Generación de score preliminar        |
| Innovación | AI copilot para recruiter             |
| Innovación | Explicabilidad del match (why fit)    |

---

## ### 4. Ideas priorizadas

| Nombre                 | Descripción                 | Actor       |
| ---------------------- | --------------------------- | ----------- |
| AI Resume Summarizer   | Resume CV en insights clave | Recruiter   |
| AI Candidate Matching  | Score automático vs job     | Recruiter   |
| AI Interview Assistant | Genera preguntas y feedback | Interviewer |

---

## ### 5. Casos de uso candidatos

* "Resumir CV automáticamente" → System → Módulo: Candidate
* "Sugerir match candidato-job" → System → Módulo: Applications
* "Generar preguntas de entrevista" → Interviewer → Módulo: Interview

---

## ### 6. Impacto en el modelo de datos

| Entidad     | Cambio                             | Tipo          |
| ----------- | ---------------------------------- | ------------- |
| Candidate   | campo `ai_summary`                 | Cambio menor  |
| Application | campo `ai_score`, `ai_explanation` | Cambio menor  |
| Interview   | campo `ai_questions`               | Cambio menor  |
| —           | entidad `AIProcessingLog`          | Entidad nueva |

---

# 🚀 Resumen Ejecutivo (Top 5 ideas más importantes)

Estas son las **5 apuestas más fuertes para el siguiente ciclo (alto impacto + MVP-friendly):**

1. **AI Candidate Matching + Score**

   * Core diferenciador inmediato
   * Impacta directamente time-to-hire

2. **Candidate Snapshot View**

   * Reduce fricción cognitiva brutalmente
   * UX win inmediato

3. **Structured Scorecards**

   * Mejora calidad de decisiones
   * Reduce sesgos

4. **Rule-Based Automation Engine**

   * Multiplicador de eficiencia
   * Base para futuras automatizaciones

5. **Candidate Comments + Mentions**

   * Desbloquea colaboración real
   * Reemplaza comunicación externa