# Funcionalidades Principales

* **Gestión de vacantes:** Crear, editar, duplicar y publicar vacantes con título, descripción, requisitos y ubicación. | **Beneficio:** Centraliza y agiliza la apertura de procesos.

* **Formulario de aplicación:** Formulario configurable para capturar datos personales, CV y respuestas básicas de screening. | **Beneficio:** Estandariza la entrada de candidatos y reduce errores.

* **Perfil de candidato:** Vista única con datos personales, CV, historial de aplicaciones, evaluaciones y actividad. | **Beneficio:** Permite seguimiento completo y decisiones informadas.

* **Pipeline de candidatos:** Tablero por etapas (Applied, Screening, Interview, etc.) con visualización tipo kanban. | **Beneficio:** Da visibilidad clara del progreso del reclutamiento.

* **Gestión de etapas:** Movimiento manual de candidatos entre etapas y definición básica del flujo por vacante. | **Beneficio:** Mantiene orden y consistencia en el proceso.

* **Evaluación y feedback:** Registro estructurado de entrevistas con puntuaciones y comentarios por entrevistador. | **Beneficio:** Reduce sesgos y mejora la calidad de contratación.

* **Notas internas:** Comentarios privados asociados al candidato visibles solo para el equipo. | **Beneficio:** Facilita la colaboración sin afectar al candidato.

* **Búsqueda y filtros:** Filtros por nombre, skills, experiencia o estado dentro del pipeline. | **Beneficio:** Acelera la identificación de candidatos relevantes.

* **Gestión de usuarios y roles:** Creación de usuarios con permisos (admin, recruiter, interviewer). | **Beneficio:** Controla accesos y protege la información.

* **Comunicación con candidatos:** Envío de emails básicos (confirmación, avance, rechazo) desde el sistema. | **Beneficio:** Mejora la experiencia del candidato y reduce trabajo manual.

# 🧭 Customer Journey – Uso de un ATS (Empresa / Customer)

---

### 1. **Descubrimiento / Awareness**

* Detecta problemas en reclutamiento (desorden, lentitud).
* Investiga soluciones (ATS, herramientas HR).
* Interactúa con marketing (web, demos, contenido).

**Relaciones:**

* Marketing ↔ HR / Founder / Hiring Manager

---

### 2. **Evaluación / Consideración**

* Compara herramientas ATS.
* Solicita demo o trial.
* Evalúa facilidad de uso, features y costo.

**Relaciones:**

* Sales ↔ HR / Decision Makers
* HR ↔ Hiring Managers (feedback interno)

---

### 3. **Compra / Decisión**

* Selecciona proveedor.
* Define plan (usuarios, vacantes).
* Firma contrato o suscripción.

**Relaciones:**

* Sales ↔ Stakeholders (HR Lead, Finance, CEO)

---

### 4. **Onboarding / Setup**

* Configura cuenta (usuarios, roles).
* Define pipeline de contratación.
* Crea primeras vacantes.

**Relaciones:**

* Customer Success ↔ HR
* HR ↔ Hiring Managers

---

### 5. **Adopción inicial (Primer uso real)**

* Publica vacantes.
* Recibe candidatos.
* Usa pipeline y mueve candidatos.

**Relaciones:**

* HR ↔ Hiring Managers
* HR ↔ Candidatos

---

### 6. **Colaboración en el proceso**

* Entrevistadores agregan feedback.
* HR coordina entrevistas.
* Equipo usa notas y evaluaciones.

**Relaciones:**

* HR ↔ Interviewers
* Hiring Managers ↔ HR
* HR ↔ Candidatos

---

### 7. **Decisión de contratación**

* Comparación de candidatos.
* Selección final.
* Envío de oferta.

**Relaciones:**

* Hiring Manager ↔ HR
* HR ↔ Candidato seleccionado

---

### 8. **Cierre del proceso**

* Contratación (Hired).
* Rechazo a otros candidatos.
* Cierre de vacante.

**Relaciones:**

* HR ↔ Candidatos
* HR ↔ Sistema ATS (registro histórico)

---

### 9. **Optimización / Uso continuo**

* Revisión de métricas (time-to-hire, funnel).
* Ajuste de pipeline.
* Reutilización de candidatos.

**Relaciones:**

* HR ↔ Leadership
* HR ↔ ATS (analytics)

---

### 10. **Expansión / Retención**

* Agrega más usuarios o vacantes.
* Integra con otras herramientas.
* Renueva suscripción.

**Relaciones:**

* Customer Success ↔ HR
* Sales ↔ Empresa (upsell)

---

## 🔁 Iteraciones clave del journey

* **Por cada nueva vacante:** vuelve a pasos 5 → 9
* **Por cada candidato:** recorre pipeline (Apply → Hired/Rejected)
* **Por cada mejora interna:** regresa a optimización (paso 9)

---

## 🔗 Relaciones principales del ecosistema

* HR ↔ Hiring Manager
* HR ↔ Candidatos
* HR ↔ Interviewers
* HR ↔ Customer Success / Sales
* Empresa ↔ ATS (plataforma como core operativo)

## CJM (mermaid)

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

