# Casos de uso esenciales para un ATS MVP

## Módulo: Gestión de Vacantes

### 1. **Crear Vacante**

* **Actor(es):** Recruiter, Admin
* **Descripción:** Permite registrar una nueva vacante con la información mínima operativa: título, área, ubicación, modalidad, descripción y estado. Este caso de uso inicia formalmente un proceso de reclutamiento dentro del sistema.
* **Precondiciones:** El usuario debe estar autenticado y contar con permisos de creación de vacantes.
* **Resultado esperado:** La vacante queda almacenada en estado borrador o abierta, lista para recibir postulaciones.
* **Prioridad:** Alta

### 2. **Editar Vacante**

* **Actor(es):** Recruiter, Admin
* **Descripción:** Permite modificar los datos de una vacante existente para corregir información o ajustar el perfil buscado. Debe preservar trazabilidad básica de cambios relevantes.
* **Precondiciones:** La vacante debe existir y el usuario debe tener permisos de edición.
* **Resultado esperado:** La vacante actualiza su información de forma consistente sin perder su relación con las postulaciones existentes.
* **Prioridad:** Alta

### 3. **Cerrar Vacante**

* **Actor(es):** Recruiter, Admin
* **Descripción:** Permite finalizar una vacante cuando ya no debe seguir recibiendo candidatos. El cierre evita nuevas postulaciones, pero conserva el historial asociado.
* **Precondiciones:** La vacante debe existir y estar activa.
* **Resultado esperado:** La vacante cambia a estado cerrada y deja de aceptar nuevas aplicaciones.
* **Prioridad:** Alta

---

## Módulo: Captura de Candidatos

### 4. **Postular a Vacante**

* **Actor(es):** Candidato
* **Descripción:** Permite al candidato completar el formulario de aplicación y adjuntar su CV para una vacante específica. Es el punto de entrada principal de datos al pipeline del ATS.
* **Precondiciones:** La vacante debe estar publicada o abierta para postulaciones.
* **Resultado esperado:** Se crea una postulación vinculada al candidato y a la vacante, con su información básica registrada.
* **Prioridad:** Alta

### 5. **Registrar Candidato Manualmente**

* **Actor(es):** Recruiter, Admin
* **Descripción:** Permite ingresar un candidato de forma manual cuando proviene de una fuente offline o externa al portal de postulación. Debe permitir asociarlo inmediatamente a una vacante.
* **Precondiciones:** El usuario debe estar autenticado y contar con permisos de gestión de candidatos.
* **Resultado esperado:** El candidato queda registrado en el sistema con una postulación activa o disponible para asignación posterior.
* **Prioridad:** Alta

---

## Módulo: Gestión de Candidatos

### 6. **Consultar Perfil de Candidato**

* **Actor(es):** Recruiter, Hiring Manager, Admin
* **Descripción:** Permite visualizar la ficha consolidada del candidato, incluyendo datos personales, CV, postulaciones y actividad relevante. Debe servir como vista central para evaluación y seguimiento.
* **Precondiciones:** El candidato debe existir y el actor debe tener permisos de lectura.
* **Resultado esperado:** El usuario accede a una vista única y consistente del candidato.
* **Prioridad:** Alta

### 7. **Buscar Candidato**

* **Actor(es):** Recruiter, Hiring Manager, Admin
* **Descripción:** Permite localizar candidatos mediante filtros básicos como nombre, email, vacante o estado del proceso. Es esencial para operar el sistema de forma eficiente cuando crece el volumen.
* **Precondiciones:** Deben existir candidatos registrados en el sistema.
* **Resultado esperado:** El sistema devuelve un conjunto filtrado de candidatos según los criterios ingresados.
* **Prioridad:** Alta

### 8. **Actualizar Información de Candidato**

* **Actor(es):** Recruiter, Admin
* **Descripción:** Permite corregir o complementar información del candidato sin alterar indebidamente el historial del proceso. Debe aplicarse sobre atributos editables definidos por negocio.
* **Precondiciones:** El candidato debe existir y el usuario debe tener permisos de edición.
* **Resultado esperado:** La ficha del candidato queda actualizada y disponible para consulta posterior.
* **Prioridad:** Media

---

## Módulo: Pipeline de Reclutamiento

### 9. **Visualizar Pipeline de Vacante**

* **Actor(es):** Recruiter, Hiring Manager, Admin
* **Descripción:** Permite ver los candidatos organizados por etapa dentro del proceso de una vacante. Debe ofrecer una vista operativa clara del avance de cada candidato.
* **Precondiciones:** La vacante debe existir y tener postulaciones asociadas.
* **Resultado esperado:** El usuario puede consultar el estado actual del pipeline por vacante.
* **Prioridad:** Alta

### 10. **Mover Candidato de Etapa**

* **Actor(es):** Recruiter, Hiring Manager
* **Descripción:** Permite avanzar o retroceder un candidato entre etapas del pipeline según la evaluación del proceso. Este caso de uso materializa el flujo principal del reclutamiento.
* **Precondiciones:** Debe existir una postulación activa y un pipeline definido para la vacante.
* **Resultado esperado:** La postulación cambia de etapa y el nuevo estado queda persistido.
* **Prioridad:** Alta

### 11. **Descartar Candidato**

* **Actor(es):** Recruiter, Hiring Manager
* **Descripción:** Permite finalizar la participación de un candidato en una vacante específica cuando no continuará en el proceso. Debe registrar el estado final de la postulación.
* **Precondiciones:** La postulación debe existir y encontrarse activa.
* **Resultado esperado:** La postulación queda marcada como descartada o rechazada, sin eliminar el historial.
* **Prioridad:** Alta

### 12. **Marcar Candidato como Contratado**

* **Actor(es):** Recruiter, Admin
* **Descripción:** Permite cerrar exitosamente el proceso para un candidato seleccionado. Este caso de uso representa el resultado de negocio principal del ATS.
* **Precondiciones:** El candidato debe estar asociado a una vacante activa y en etapa final elegible.
* **Resultado esperado:** La postulación queda en estado contratado y la vacante puede cerrarse manualmente o quedar lista para cierre.
* **Prioridad:** Alta

---

## Módulo: Evaluación y Colaboración

### 13. **Registrar Feedback de Candidato**

* **Actor(es):** Recruiter, Hiring Manager, Interviewer
* **Descripción:** Permite capturar observaciones, comentarios y valoración cualitativa sobre un candidato en una etapa del proceso. Debe quedar asociado a la postulación o etapa correspondiente.
* **Precondiciones:** El candidato debe estar en proceso y el actor debe tener permisos para comentar.
* **Resultado esperado:** El sistema almacena feedback consultable por los participantes autorizados.
* **Prioridad:** Alta

### 14. **Registrar Evaluación de Entrevista**

* **Actor(es):** Interviewer, Hiring Manager, Recruiter
* **Descripción:** Permite documentar el resultado estructurado de una entrevista mediante comentarios y una calificación básica. Su objetivo es soportar decisiones más consistentes y comparables.
* **Precondiciones:** Debe existir una entrevista realizada o una etapa de entrevista activa para la postulación.
* **Resultado esperado:** La evaluación queda registrada y disponible para revisión del equipo de contratación.
* **Prioridad:** Alta

---

## Módulo: Comunicación Operativa

### 15. **Enviar Comunicación al Candidato**

* **Actor(es):** Recruiter
* **Descripción:** Permite enviar mensajes operativos básicos al candidato, como confirmación de recepción, avance o rechazo. Para MVP debe limitarse a plantillas simples o mensajes manuales desde el sistema.
* **Precondiciones:** El candidato debe tener email registrado y estar asociado a una postulación.
* **Resultado esperado:** La comunicación queda enviada y, idealmente, registrada en el historial de la postulación.
* **Prioridad:** Alta

---

## Módulo: Administración Básica

### 16. **Gestionar Usuarios**

* **Actor(es):** Admin
* **Descripción:** Permite crear, editar y desactivar usuarios internos del ATS. Es necesario para controlar quién participa en el proceso y con qué nivel de acceso.
* **Precondiciones:** El actor debe tener rol administrativo.
* **Resultado esperado:** Los usuarios internos quedan habilitados o actualizados con sus credenciales y estado operativo.
* **Prioridad:** Alta

### 17. **Asignar Roles y Permisos Básicos**

* **Actor(es):** Admin
* **Descripción:** Permite definir el nivel de acceso de cada usuario según su función operativa, por ejemplo Admin, Recruiter o Hiring Manager. En MVP debe resolverse con un esquema simple y estable de roles.
* **Precondiciones:** El usuario interno debe existir en el sistema.
* **Resultado esperado:** Cada usuario queda asociado a un rol válido y sus permisos son aplicados en la interfaz y lógica de negocio.
* **Prioridad:** Alta

---

# Resumen de alcance MVP

## Prioridad Alta

* Crear Vacante
* Editar Vacante
* Cerrar Vacante
* Postular a Vacante
* Registrar Candidato Manualmente
* Consultar Perfil de Candidato
* Buscar Candidato
* Visualizar Pipeline de Vacante
* Mover Candidato de Etapa
* Descartar Candidato
* Marcar Candidato como Contratado
* Registrar Feedback de Candidato
* Registrar Evaluación de Entrevista
* Enviar Comunicación al Candidato
* Gestionar Usuarios
* Asignar Roles y Permisos Básicos


## UML Diagrams

### Gestión de Vacantes
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