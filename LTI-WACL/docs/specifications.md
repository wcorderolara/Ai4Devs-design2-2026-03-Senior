# Specification-Driven Development (SDD) — ATS Multi-Tenant

**Proyecto:** Applicant Tracking System (ATS) SaaS Multi-Tenant
**Versión del documento:** 1.0.0
**Última actualización:** 2026-04-20
**Owner:** Walter
**Audiencia primaria:** Agente de IA (generador de código) + desarrolladores humanos

---

## 📑 Tabla de Contenidos

1. [Meta y Propósito del Documento](#1-meta-y-propósito-del-documento)
2. [Reglas No-Negociables Globales](#2-reglas-no-negociables-globales)
3. [Stack Tecnológico Fijado](#3-stack-tecnológico-fijado)
4. [Arquitectura y Bounded Contexts](#4-arquitectura-y-bounded-contexts)
5. [Estructura de Carpetas](#5-estructura-de-carpetas)
6. [Convenciones de Naming](#6-convenciones-de-naming)
7. [Multi-Tenancy](#7-multi-tenancy)
8. [Autenticación y RBAC](#8-autenticación-y-rbac)
9. [Capa de Dominio (DDD)](#9-capa-de-dominio-ddd)
10. [Capa de Aplicación (Use Cases + CQRS ligero)](#10-capa-de-aplicación-use-cases--cqrs-ligero)
11. [Capa de Infraestructura](#11-capa-de-infraestructura)
12. [API REST](#12-api-rest)
13. [Validación y Manejo de Errores](#13-validación-y-manejo-de-errores)
14. [Frontend (Next.js)](#14-frontend-nextjs)
15. [Testing](#15-testing)
16. [Logging, Auditoría y Observabilidad](#16-logging-auditoría-y-observabilidad)
17. [Git Workflow](#17-git-workflow)
18. [Checklist del Agente](#18-checklist-del-agente)
19. [Anti-Patrones Prohibidos](#19-anti-patrones-prohibidos)
20. [Templates Copy-Paste: Caso de Uso Completo "Crear Vacante"](#20-templates-copy-paste-caso-de-uso-completo-crear-vacante)

---

## 1. Meta y Propósito del Documento

### 1.1 Qué es este documento

Este documento es el **contrato técnico vinculante** que el Agente de IA DEBE seguir al generar, modificar o refactorizar código de este proyecto. No es una guía de estilo sugerida. No es una referencia opcional. Es una especificación ejecutable.

### 1.2 Cómo debe usarlo el Agente de IA

**ANTES de ejecutar cualquier tarea, el agente DEBE:**

1. Leer la sección relevante de este documento.
2. Identificar las reglas MUST y MUST NOT que aplican.
3. Validar que la tarea solicitada no contradice las reglas. Si contradice, el agente DEBE detenerse y pedir clarificación al usuario.
4. Ejecutar siguiendo los templates de la sección 20 como referencia exacta.
5. Al finalizar, ejecutar el Checklist del Agente (sección 18).

### 1.3 Prioridad de reglas

El orden de precedencia cuando hay conflicto es:

1. **MUST / MUST NOT de este documento** (máxima prioridad).
2. Instrucciones explícitas del usuario en la conversación actual.
3. Convenciones estándar del ecosistema (Next.js, Prisma, etc.).
4. Preferencias inferidas del código existente.

**El agente NUNCA debe inferir contra una regla MUST.** Si el usuario pide algo que viola una regla MUST, el agente DEBE informar el conflicto y pedir confirmación explícita antes de proceder.

### 1.4 Terminología RFC 2119

Este documento usa las palabras clave **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, **MAY** según [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119):

- **MUST / DEBE:** Requerimiento absoluto. Violarlo es un error.
- **MUST NOT / NO DEBE:** Prohibición absoluta.
- **SHOULD / DEBERÍA:** Recomendación fuerte. Puede desviarse con justificación escrita.
- **SHOULD NOT / NO DEBERÍA:** Desaconsejado. Puede hacerse con justificación.
- **MAY / PUEDE:** Opcional.

### 1.5 Versionado de este documento

Cambios en este documento siguen SemVer:
- **MAJOR:** ruptura arquitectónica (ej: cambio de ORM).
- **MINOR:** nuevas reglas o secciones.
- **PATCH:** clarificaciones, correcciones tipográficas.

---

## 2. Reglas No-Negociables Globales

Estas reglas aplican a **todo el código** del proyecto sin excepción.

### 2.1 Lenguaje y Tipado

- **MUST** usar TypeScript en todo el proyecto (backend y frontend).
- **MUST** usar `strict: true` en todos los `tsconfig.json`.
- **MUST NOT** usar el tipo `any` bajo ninguna circunstancia. Alternativas aceptadas: `unknown`, tipo específico, genérico, `never`.
- **MUST NOT** usar `@ts-ignore`, `@ts-nocheck` ni `@ts-expect-error` sin un comentario justificando en la línea siguiente con formato: `// JUSTIFICACIÓN: <razón>` y marcado con ticket de seguimiento.
- **MUST** usar `readonly` en propiedades de clases que no se muten después del constructor.
- **MUST** preferir `type` para tipos simples y `interface` para contratos extensibles/implementables.
- **MUST** usar ES Modules (`import`/`export`), nunca CommonJS (`require`/`module.exports`).
- **MUST NOT** usar `var`. Usar `const` por defecto; `let` solo cuando hay reasignación real.

### 2.2 Determinismo y Pureza

- **MUST** aislar side-effects en la capa de Infraestructura. Las capas de Dominio y Aplicación DEBEN ser puras (salvo uso explícito de puertos inyectados).
- **MUST NOT** leer variables de entorno (`process.env`) fuera del módulo de configuración (`packages/config` o `apps/api/src/config/`).
- **MUST NOT** usar `Date.now()` o `new Date()` directamente en Dominio o Aplicación. Usar un puerto `Clock` inyectado.
- **MUST NOT** usar `Math.random()` o `crypto.randomUUID()` directamente en Dominio o Aplicación. Usar un puerto `IdGenerator`.

### 2.3 Inmutabilidad

- **MUST** tratar DTOs, Value Objects y eventos como inmutables.
- **SHOULD** usar `Readonly<T>`, `ReadonlyArray<T>` en firmas públicas.
- **MUST NOT** mutar parámetros de función.

### 2.4 Errores

- **MUST** usar la jerarquía de errores definida en la sección 13. Nunca lanzar `new Error(...)` genérico en capas de Dominio o Aplicación.
- **MUST NOT** usar `throw` de strings o números. Solo instancias que extienden `BaseError`.
- **MUST** capturar errores en la capa más externa (middleware de Express / error boundary de Next.js). Las capas internas DEBEN dejar propagar.

### 2.5 Multi-tenancy

- **MUST** incluir `tenantId` en **TODA** operación que acceda a datos del dominio del tenant.
- **MUST NOT** confiar en el `tenantId` enviado en el body de la request. SIEMPRE usar el `tenantId` resuelto desde el contexto de la request (path param validado + sesión).
- **MUST** verificar que el `tenantId` del path coincide con el `tenantId` del header `X-Tenant-ID` y con el `tenantId` de la sesión del usuario autenticado.

### 2.6 Seguridad

- **MUST NOT** loguear secretos, tokens, contraseñas, ni PII sin redacción explícita.
- **MUST** usar parámetros preparados en queries SQL (Prisma ya lo garantiza; si se escribe SQL crudo, DEBE usar `$queryRaw` con template strings, NUNCA concatenación).
- **MUST NOT** exponer detalles internos (stack traces, nombres de tablas) en respuestas de error a clientes en producción.

### 2.7 Formato y Linting

- **MUST** pasar `eslint` y `prettier` sin errores antes de commit.
- **MUST** mantener indentación de 2 espacios.
- **MUST** usar comillas simples en TypeScript (`'`), backticks para templates.
- **MUST** terminar statements con punto y coma.
- **MUST** tener trailing comma en multiline.
- **MUST NOT** exceder 100 caracteres por línea (soft limit 80).

### 2.8 Comentarios

- **MUST NOT** escribir comentarios redundantes que repiten el código.
- **SHOULD** escribir comentarios explicando **el porqué**, no **el qué**.
- **MUST** usar JSDoc en métodos públicos exportados de packages compartidos.
- **MUST** marcar TODOs con formato: `// TODO(<ticket-id>): <descripción>` — TODOs sin ticket NO están permitidos.

---

## 3. Stack Tecnológico Fijado

### 3.1 Versiones exactas

El agente **MUST NOT** introducir paquetes ni versiones que no estén listados aquí. Para agregar una dependencia nueva, el agente DEBE pedir aprobación explícita.

| Categoría | Tecnología | Versión mínima |
|---|---|---|
| Runtime | Node.js | 20.x LTS |
| Package manager | pnpm | 9.x |
| Monorepo | Turborepo | 2.x |
| Lenguaje | TypeScript | 5.4+ |
| Backend framework | Express | 4.19+ |
| ORM | Prisma | 5.x |
| Validación | Zod | 3.23+ |
| Base de datos | PostgreSQL | 16+ |
| Cola de tareas | BullMQ | 5.x |
| Broker (colas) | Redis | 7.x |
| Logging | Pino | 9.x |
| Tracing | OpenTelemetry | 1.x |
| Error tracking | Sentry | 8.x |
| Frontend framework | Next.js | 14.2+ |
| UI components | shadcn/ui | latest |
| Styling | Tailwind CSS | 3.4+ |
| Auth | Auth.js (NextAuth) | v5 beta |
| Client state | Zustand | 4.5+ |
| Server state | TanStack Query | 5.x |
| Forms | React Hook Form | 7.x |
| Testing unit/integration | Jest | 29.x |
| Testing e2e | Playwright | 1.44+ |
| Deployment | AWS ECS Fargate + RDS Postgres | — |

### 3.2 Paquetes explícitamente prohibidos

El agente **MUST NOT** usar los siguientes paquetes:

- `moment` → usar `date-fns` o `dayjs`.
- `lodash` completo → usar `lodash-es` con imports específicos o nativos.
- `axios` → usar `fetch` nativo (Node 20 lo soporta).
- `body-parser` (separado) → usar `express.json()` / `express.urlencoded()`.
- `jsonwebtoken` → manejado por Auth.js.
- `bcrypt` (nativo) → usar `bcryptjs` (cross-platform, evita issues en Docker ARM).
- `uuid` → usar `crypto.randomUUID()` vía el puerto `IdGenerator`.
- `class-validator` / `class-transformer` → usar Zod.
- `typeorm` / `sequelize` / `mongoose` → prohibido (stack es Prisma).
- Cualquier librería de UI que no sea Tailwind + shadcn/ui (no MUI, no Chakra, no Ant Design).

### 3.3 Gestión de dependencias

- **MUST** usar `pnpm` (nunca `npm` ni `yarn`).
- **MUST** fijar versiones con `~` para parches (`~5.4.0`) — no usar `^` en producción.
- **MUST** ejecutar `pnpm audit` antes de cada release.
- **MUST** usar `pnpm-workspace.yaml` en la raíz.

---

## 4. Arquitectura y Bounded Contexts

### 4.1 Estilo arquitectónico

El backend es un **Modular Monolith** con los siguientes principios no-negociables:

- Cada bounded context es un módulo independiente en `apps/api/src/modules/<context>/`.
- Los módulos **MUST NOT** importar código de otros módulos directamente. Comunicación SOLO vía:
  1. **Domain Events** (para comunicación asíncrona in-process).
  2. **Application-level Contracts** (interfaces públicas explícitas expuestas en `index.ts` del módulo).
- Cada módulo DEBE poder extraerse a un microservicio sin reescribir lógica de dominio.

### 4.2 Bounded Contexts del ATS

| Bounded Context | Responsabilidad | Aggregates principales |
|---|---|---|
| `iam` | Identity & Access Management: autenticación, autorización, gestión de usuarios, RBAC | `User`, `Role`, `Permission`, `Session` |
| `tenancy` | Gestión de tenants (empresas cliente), configuración, branding | `Tenant`, `TenantSettings` |
| `jobs` | Creación y gestión del ciclo de vida de vacantes | `Job` |
| `candidates` | Gestión del perfil de candidatos (datos personales, CV, historial) | `Candidate` |
| `applications` | Proceso de aplicación: relación entre `Candidate` y `Job`, estados del pipeline | `Application` |
| `interviews` | Agendamiento y gestión de entrevistas | `Interview` |
| `notifications` | Envío de emails y notificaciones (reacciona a eventos) | `NotificationRequest` |
| `audit` | Registro inmutable de acciones sensibles para cumplimiento | `AuditLog` |

### 4.3 Diagrama C4 — Nivel de Contenedores

```
┌───────────────────────────────────────────────────────────────────┐
│                           USUARIO (Browser)                        │
└──────────────────┬─────────────────────────────┬──────────────────┘
                   │ HTTPS                        │ HTTPS
                   ▼                              ▼
       ┌───────────────────────┐     ┌──────────────────────────────┐
       │   Next.js 14 (ECS)    │◄────┤  ALB (AWS Application Load   │
       │  App Router + RSC     │     │  Balancer con WAF)           │
       │  Server Actions       │     └──────────────────────────────┘
       └───────────┬───────────┘
                   │ REST/JSON (interno VPC)
                   ▼
       ┌───────────────────────┐
       │  Express API (ECS)    │◄────── BullMQ Workers (ECS, mismo imagen)
       │  Modular Monolith     │
       │  DDD + CQRS ligero    │
       └───┬───────┬───────┬───┘
           │       │       │
           ▼       ▼       ▼
    ┌─────────┐ ┌────────┐ ┌────────────┐
    │   RDS   │ │ Redis  │ │   S3       │
    │ Postgres│ │ElastiC │ │ (CVs,      │
    │   16    │ │ 7.x    │ │  exports)  │
    └─────────┘ └────────┘ └────────────┘
           │
           ▼
    ┌─────────────────────┐
    │  Sentry + Otel +    │
    │  CloudWatch Logs    │
    └─────────────────────┘
```

### 4.4 Capas dentro de cada módulo (Clean Architecture)

Cada bounded context DEBE tener exactamente estas cuatro capas, en este orden de dependencias (de dentro hacia afuera):

```
domain/         ← Reglas de negocio puras. NO depende de nada externo.
application/    ← Casos de uso. Depende SOLO de domain/ y de puertos.
infrastructure/ ← Implementaciones concretas de puertos (Prisma, Redis, etc.).
interface/      ← Adaptadores de entrada (controllers HTTP, workers, jobs).
```

**Regla de dependencias (MUST):**

- `domain/` **MUST NOT** importar de ninguna otra capa.
- `application/` **MUST** importar solo de `domain/` (y puertos definidos allí).
- `infrastructure/` **MUST** importar de `domain/` y `application/` (implementa sus puertos).
- `interface/` **MUST** importar de `application/`. **MUST NOT** importar directamente de `domain/`.

El agente **MUST** verificar estas reglas con un linter (dependency-cruiser) antes de cada commit.

### 4.5 CQRS ligero

Separación de **intención**, no de infraestructura:

- **Commands** modifican estado. Viven en `application/commands/`. Retornan `void` o un ID generado.
- **Queries** leen estado. Viven en `application/queries/`. Retornan DTOs de lectura.
- Ambos usan el **mismo modelo de dominio** y la **misma base de datos** (no read-models separados).
- **MUST NOT** mezclar lectura y escritura en el mismo handler.
- **SHOULD** permitir que las queries hagan consultas optimizadas (joins, proyecciones) saltando el aggregate cuando la performance lo justifique — esto es aceptable en CQRS ligero.

---

## 5. Estructura de Carpetas

### 5.1 Raíz del monorepo

```
ats-saas/
├── .github/
│   └── workflows/              # CI/CD pipelines (GitHub Actions)
├── .vscode/
│   └── settings.json           # Configuración compartida del editor
├── apps/
│   ├── api/                    # Backend Express (Modular Monolith)
│   └── web/                    # Frontend Next.js
├── packages/
│   ├── contracts/              # Zod schemas + tipos DTO compartidos
│   ├── ui/                     # Componentes React reutilizables (shadcn/ui)
│   ├── config-ts/              # Configs base de tsconfig
│   ├── config-eslint/          # Configs base de ESLint
│   ├── config-tailwind/        # Preset de Tailwind
│   └── logger/                 # Wrapper de Pino + Otel compartido
├── infra/
│   ├── docker/                 # Dockerfiles y compose
│   ├── terraform/              # IaC para AWS
│   └── migrations/             # Migraciones SQL consolidadas (si se exportan)
├── scripts/                    # Scripts de utilidad (seed, migrate, etc.)
├── docs/
│   ├── SDD.md                  # ESTE documento
│   ├── ADRs/                   # Architecture Decision Records
│   └── runbooks/               # Procedimientos operativos
├── .gitignore
├── .nvmrc                      # Fijar versión Node
├── .editorconfig
├── package.json                # Root (scripts de workspace)
├── pnpm-workspace.yaml
├── pnpm-lock.yaml
├── turbo.json
├── tsconfig.base.json
└── README.md
```

### 5.2 Estructura de `apps/api/`

```
apps/api/
├── src/
│   ├── main.ts                 # Entry point (bootstrap Express)
│   ├── app.ts                  # Construcción de la app Express (sin listen)
│   ├── config/
│   │   ├── index.ts            # Config validada con Zod
│   │   ├── env.schema.ts       # Schema Zod de variables de entorno
│   │   └── constants.ts
│   ├── shared/                 # Código compartido entre módulos (SIN lógica de dominio)
│   │   ├── domain/
│   │   │   ├── errors/         # BaseError, DomainError, ValidationError
│   │   │   ├── ports/          # Clock, IdGenerator, EventBus
│   │   │   └── value-objects/  # VOs genéricos (Email, PhoneNumber)
│   │   ├── infrastructure/
│   │   │   ├── prisma/         # PrismaClient singleton
│   │   │   ├── redis/
│   │   │   ├── bullmq/
│   │   │   ├── logger/
│   │   │   ├── otel/
│   │   │   ├── clock/
│   │   │   └── id-generator/
│   │   └── interface/
│   │       ├── middleware/     # auth, tenant-resolver, error-handler, request-id
│   │       └── http/           # Tipos compartidos Express
│   ├── modules/
│   │   ├── iam/
│   │   ├── tenancy/
│   │   ├── jobs/
│   │   ├── candidates/
│   │   ├── applications/
│   │   ├── interviews/
│   │   ├── notifications/
│   │   └── audit/
│   └── workers/                # BullMQ workers entry point
│       ├── main.ts
│       └── processors/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── test/
│   ├── setup/
│   ├── fixtures/
│   └── integration/            # Tests que tocan BD real (testcontainers)
├── jest.config.ts
├── jest.integration.config.ts
├── tsconfig.json
├── package.json
└── Dockerfile
```

### 5.3 Estructura de un módulo (ejemplo: `jobs/`)

**Esta estructura es OBLIGATORIA y DEBE replicarse idénticamente en todos los módulos.**

```
modules/jobs/
├── domain/
│   ├── entities/
│   │   └── job.entity.ts           # Aggregate root
│   ├── value-objects/
│   │   ├── job-id.vo.ts
│   │   ├── job-title.vo.ts
│   │   ├── job-status.vo.ts
│   │   └── job-location.vo.ts
│   ├── events/
│   │   ├── job-created.event.ts
│   │   └── job-published.event.ts
│   ├── errors/
│   │   └── job.errors.ts           # Errores específicos del dominio
│   ├── ports/
│   │   └── job.repository.ts       # Interface (puerto de salida)
│   └── policies/
│       └── job-visibility.policy.ts
├── application/
│   ├── commands/
│   │   ├── create-job/
│   │   │   ├── create-job.command.ts       # DTO de entrada
│   │   │   ├── create-job.handler.ts       # Orquestación
│   │   │   └── create-job.handler.spec.ts  # Unit test del handler
│   │   └── publish-job/
│   ├── queries/
│   │   ├── get-job-by-id/
│   │   │   ├── get-job-by-id.query.ts
│   │   │   ├── get-job-by-id.handler.ts
│   │   │   └── get-job-by-id.handler.spec.ts
│   │   └── list-jobs/
│   ├── dtos/
│   │   ├── job.dto.ts                      # DTO de salida (read model)
│   │   └── job-summary.dto.ts
│   └── ports/
│       └── (puertos de aplicación adicionales, ej: NotificationService)
├── infrastructure/
│   ├── persistence/
│   │   ├── prisma-job.repository.ts        # Implementa JobRepository
│   │   └── job.mapper.ts                   # Domain ↔ Prisma model
│   └── (otras implementaciones: HTTP clients externos, etc.)
├── interface/
│   ├── http/
│   │   ├── jobs.controller.ts
│   │   ├── jobs.routes.ts
│   │   └── dtos/
│   │       ├── create-job.request.ts       # Zod schema de request
│   │       └── job.response.ts             # Zod schema de response
│   └── events/
│       └── (event handlers que reaccionan a eventos de otros módulos)
├── jobs.module.ts                          # Composition root del módulo
├── index.ts                                # Exportaciones públicas
└── __tests__/
    └── integration/
        └── create-job.integration.spec.ts
```

### 5.4 Estructura de `apps/web/` (Next.js App Router)

```
apps/web/
├── src/
│   ├── app/
│   │   ├── layout.tsx                      # Root layout
│   │   ├── page.tsx                        # Landing pública
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── t/
│   │   │   └── [tenantSlug]/
│   │   │       ├── layout.tsx              # Layout con tenant resolver
│   │   │       ├── page.tsx                # Dashboard del tenant
│   │   │       ├── jobs/
│   │   │       │   ├── page.tsx            # Lista de vacantes
│   │   │       │   ├── new/
│   │   │       │   │   └── page.tsx        # Form crear vacante
│   │   │       │   └── [jobId]/
│   │   │       │       ├── page.tsx
│   │   │       │       └── edit/
│   │   │       │           └── page.tsx
│   │   │       ├── candidates/
│   │   │       ├── applications/
│   │   │       └── interviews/
│   │   └── api/
│   │       └── auth/
│   │           └── [...nextauth]/
│   │               └── route.ts
│   ├── features/
│   │   ├── jobs/
│   │   │   ├── components/
│   │   │   │   ├── job-list.tsx
│   │   │   │   ├── job-form.tsx
│   │   │   │   └── job-card.tsx
│   │   │   ├── hooks/
│   │   │   │   ├── use-jobs.ts             # TanStack Query hooks
│   │   │   │   └── use-create-job.ts       # Mutation hook
│   │   │   ├── actions/
│   │   │   │   └── create-job.action.ts    # Server Action
│   │   │   ├── schemas/
│   │   │   │   └── job.schema.ts           # Re-export de @ats/contracts
│   │   │   └── types/
│   │   │       └── job.types.ts
│   │   ├── candidates/
│   │   ├── applications/
│   │   └── iam/
│   ├── components/
│   │   ├── ui/                             # shadcn/ui generated
│   │   ├── layout/
│   │   │   ├── header.tsx
│   │   │   ├── sidebar.tsx
│   │   │   └── tenant-switcher.tsx
│   │   └── shared/
│   ├── lib/
│   │   ├── api-client.ts                   # Fetch wrapper con auth + tenant
│   │   ├── auth.ts                         # Config de Auth.js
│   │   ├── query-client.ts                 # TanStack Query setup
│   │   ├── utils.ts                        # cn(), formatters
│   │   └── permissions.ts                  # Checks RBAC client-side
│   ├── stores/
│   │   ├── ui.store.ts                     # Zustand: UI global
│   │   └── tenant.store.ts                 # Zustand: tenant activo
│   ├── providers/
│   │   ├── query-provider.tsx
│   │   ├── theme-provider.tsx
│   │   └── session-provider.tsx
│   ├── middleware.ts                       # Next.js middleware (auth guard)
│   └── types/
│       └── global.d.ts
├── public/
├── e2e/                                    # Playwright
│   ├── fixtures/
│   ├── specs/
│   └── playwright.config.ts
├── tailwind.config.ts
├── next.config.ts
├── tsconfig.json
└── package.json
```

### 5.5 Reglas de estructura

- **MUST** replicar exactamente la estructura de módulos mostrada en 5.3.
- **MUST NOT** crear carpetas con nombres diferentes (`controllers/` en lugar de `interface/http/`, `models/` en lugar de `entities/`).
- **MUST** mantener un `index.ts` en la raíz de cada módulo que exporta solo lo público.
- **MUST NOT** importar de paths profundos de otros módulos. Solo desde `modules/<otro>/index.ts`.
- **MUST** colocar tests unitarios `.spec.ts` junto al archivo que testean.
- **MUST** colocar tests de integración en `__tests__/integration/` dentro del módulo.

---

## 6. Convenciones de Naming

### 6.1 Archivos

| Tipo | Patrón | Ejemplo |
|---|---|---|
| Entity (aggregate) | `<name>.entity.ts` | `job.entity.ts` |
| Value Object | `<name>.vo.ts` | `job-title.vo.ts` |
| Domain Event | `<past-tense-verb>.event.ts` | `job-created.event.ts` |
| Repository interface | `<name>.repository.ts` | `job.repository.ts` |
| Repository impl | `prisma-<name>.repository.ts` | `prisma-job.repository.ts` |
| Mapper | `<name>.mapper.ts` | `job.mapper.ts` |
| Command | `<verb>-<name>.command.ts` | `create-job.command.ts` |
| Command Handler | `<verb>-<name>.handler.ts` | `create-job.handler.ts` |
| Query | `<verb>-<name>.query.ts` | `get-job-by-id.query.ts` |
| Controller | `<name>.controller.ts` | `jobs.controller.ts` |
| Routes | `<name>.routes.ts` | `jobs.routes.ts` |
| Zod schema | `<name>.schema.ts` | `job.schema.ts` |
| Request DTO | `<verb>-<name>.request.ts` | `create-job.request.ts` |
| Response DTO | `<name>.response.ts` | `job.response.ts` |
| Domain Error | `<module>.errors.ts` | `job.errors.ts` |
| Policy | `<name>.policy.ts` | `job-visibility.policy.ts` |
| Port / Interface | `<name>.port.ts` or `<name>.repository.ts` | `clock.port.ts` |
| Unit test | `<sibling>.spec.ts` | `create-job.handler.spec.ts` |
| Integration test | `<feature>.integration.spec.ts` | `create-job.integration.spec.ts` |
| E2E test | `<feature>.e2e.spec.ts` | `job-creation-flow.e2e.spec.ts` |
| React component | `<kebab-name>.tsx` | `job-form.tsx` |
| React hook | `use-<kebab-name>.ts` | `use-jobs.ts` |
| Zustand store | `<name>.store.ts` | `tenant.store.ts` |
| Server Action | `<verb>-<name>.action.ts` | `create-job.action.ts` |

### 6.2 Convenciones por tipo de identificador

| Categoría | Caso | Ejemplo |
|---|---|---|
| Variables, funciones, parámetros | `camelCase` | `jobTitle`, `findJobById` |
| Clases, interfaces, types | `PascalCase` | `Job`, `JobRepository`, `JobStatus` |
| Constantes globales | `SCREAMING_SNAKE_CASE` | `MAX_JOB_TITLE_LENGTH` |
| Enums | `PascalCase` (nombre) + `PascalCase` (miembros) | `JobStatus.Draft` |
| Archivos | `kebab-case` | `create-job.handler.ts` |
| Carpetas | `kebab-case` | `bounded-contexts/` |
| React Components | `PascalCase` (nombre del componente), archivo `kebab-case` | Componente `JobForm` en `job-form.tsx` |
| Hooks | `usePascalCase` (nombre), archivo `use-kebab-case.ts` | `useJobs()` en `use-jobs.ts` |
| Server Actions | `camelCase` con sufijo `Action` | `createJobAction` |
| Event classes | `PascalCase` con sufijo `Event` | `JobCreatedEvent` |
| Commands | `PascalCase` con sufijo `Command` | `CreateJobCommand` |
| Queries | `PascalCase` con sufijo `Query` | `GetJobByIdQuery` |
| Handlers | `PascalCase` con sufijo `Handler` | `CreateJobHandler` |

### 6.3 Base de datos (PostgreSQL)

- **MUST** usar `snake_case` para tablas y columnas.
- **MUST** usar plural para tablas (`jobs`, no `job`).
- **MUST** usar sufijo `_id` para foreign keys (`tenant_id`, `job_id`).
- **MUST** usar sufijo `_at` para timestamps (`created_at`, `updated_at`, `deleted_at`).
- **MUST** usar sufijo `_by` para referencias de autoría (`created_by`, `updated_by`).
- **MUST** usar prefijo `is_` o `has_` para booleanos (`is_active`, `has_completed_profile`).
- **MUST** nombrar índices con patrón: `idx_<table>_<columns>` (ej: `idx_jobs_tenant_id_status`).
- **MUST** nombrar unique constraints: `uq_<table>_<columns>`.
- **MUST** nombrar foreign keys: `fk_<table>_<referenced_table>`.

Como Prisma usa camelCase en el modelo, **MUST** mapear con `@@map()` y `@map()`:

```prisma
model Job {
  id        String   @id @default(cuid())
  tenantId  String   @map("tenant_id")
  title     String
  createdAt DateTime @default(now()) @map("created_at")

  @@map("jobs")
  @@index([tenantId, status], map: "idx_jobs_tenant_id_status")
}
```

### 6.4 URLs y endpoints

- **MUST** usar `kebab-case` en paths: `/api/v1/t/{tenantSlug}/job-postings`, no `/jobPostings`.
- **MUST** usar plural para colecciones de recursos: `/jobs`, no `/job`.
- **MUST** versionar en el path: `/api/v1/...`.
- **MUST NOT** incluir verbos en el path (usar HTTP methods). Excepción: acciones que no son CRUD (`/jobs/{id}/publish`).

### 6.5 Branches y commits

Ver sección 17.

---

## 7. Multi-Tenancy

### 7.1 Modelo de aislamiento

Este proyecto usa **shared schema + `tenant_id` column** en TODAS las tablas que contienen datos del dominio del tenant.

- **MUST** agregar columna `tenant_id` (tipo `TEXT`, FK a `tenants.id`) en cada tabla que almacene datos pertenecientes a un tenant.
- **MUST** indexar `tenant_id` en cada tabla.
- **MUST** incluir `tenant_id` como primera columna en índices compuestos que filtren por tenant + otros campos.

**Tablas SIN `tenant_id` (globales):**

- `tenants` (el tenant mismo).
- `platform_admins` (super-admins de la plataforma, no de un tenant).
- `system_events` (eventos de sistema no-tenant).

### 7.2 Resolución del tenant en requests

El tenant se identifica mediante **doble canal redundante**:

1. **Path parameter:** `/api/v1/t/:tenantSlug/...` — legible, shareable, cacheable.
2. **Header custom:** `X-Tenant-ID` — anti-CSRF, validación cruzada.

**Flujo de resolución (MUST seguirse exactamente):**

```
1. Request llega a Express.
2. Middleware `requestId` asigna un correlation ID.
3. Middleware `auth` valida sesión (Auth.js) → extrae userId + sessionTenantIds.
4. Middleware `tenantResolver`:
   a. Extrae `:tenantSlug` del path. Si no existe → 400.
   b. Extrae `X-Tenant-ID` del header. Si no existe → 400.
   c. Busca el tenant por slug en BD (con cache Redis, TTL 60s).
   d. Verifica que `tenant.id === X-Tenant-ID`. Si no coincide → 403 TENANT_MISMATCH.
   e. Verifica que el `userId` tiene acceso al tenant (tabla `user_tenants`). Si no → 403.
   f. Inyecta `req.tenantId`, `req.tenantSlug`, `req.user` en la request.
5. Middleware `rbac` verifica permisos específicos del endpoint.
6. Controller ejecuta.
```

### 7.3 Reglas de código multi-tenant

- **MUST** pasar `tenantId` como primer parámetro (o dentro de un contexto `TenantContext`) a TODO use case, command handler, query handler y repository method.
- **MUST** filtrar por `tenant_id` en TODA query Prisma. Sin excepciones.
- **MUST NOT** confiar en `tenantId` enviado en body. SIEMPRE usar el del `TenantContext`.
- **MUST** validar en repositorios que el aggregate recuperado pertenece al tenant solicitado. Si no → `TenantMismatchError`.
- **MUST** incluir `tenantId` en todos los logs estructurados.
- **MUST** incluir `tenantId` en todos los domain events.

### 7.4 Patrón `TenantContext`

```typescript
// apps/api/src/shared/domain/tenant-context.ts
export interface TenantContext {
  readonly tenantId: string;
  readonly tenantSlug: string;
  readonly userId: string;
  readonly userRoles: readonly string[];
  readonly requestId: string;
}
```

**Todo use case DEBE recibir un `TenantContext` como primer argumento.**

### 7.5 Rutas que NO llevan tenant

Rutas globales (login, health, signup de nuevos tenants) NO llevan `/t/:tenantSlug/`:

- `GET /api/v1/health`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/signup` (creación de nuevo tenant)
- `GET /api/v1/tenants/by-slug/:slug` (descubrimiento de tenant)

Estas rutas **MUST NOT** permitir el acceso a datos de tenants existentes sin autenticación y verificación adicional.

---

## 8. Autenticación y RBAC

### 8.1 Autenticación

- **MUST** usar Auth.js (NextAuth v5) en el frontend como única fuente de sesiones de usuario.
- **MUST** usar el adaptador `@auth/prisma-adapter` para persistir sesiones en la misma BD Postgres.
- **MUST** configurar sesiones tipo JWT (stateless) con expiración corta (15 minutos) + refresh token rotativo (7 días).
- **MUST** usar cookies `httpOnly`, `secure`, `sameSite: lax`.
- **MUST NOT** implementar auth custom. Usar providers de Auth.js (Credentials, Google, Microsoft, SAML vía plugin).

### 8.2 Flujo de autenticación entre Next.js y Express

El frontend Next.js obtiene la sesión via Auth.js. Para llamar al backend Express:

1. Next.js Server Action / Server Component obtiene la sesión con `auth()`.
2. Extrae del JWT: `userId`, `allowedTenants`.
3. Firma un token corto (JWT HS256, TTL 5 minutos) con una clave compartida entre `web` y `api` (variable de entorno `INTERNAL_JWT_SECRET`).
4. Llama al backend con:
   - `Authorization: Bearer <internal-jwt>`
   - `X-Tenant-ID: <tenantId>`
   - Path: `/api/v1/t/:tenantSlug/...`
5. El backend valida el JWT, el header y el path (sección 7.2).

**MUST NOT** enviar el JWT de la sesión de Auth.js directamente al backend Express. Usar el internal JWT firmado por el web server.

### 8.3 Modelo RBAC

**Estructura en BD:**

```
tenants
users
user_tenants          (user_id, tenant_id, is_active)
roles                 (id, tenant_id, name, is_system)         -- NULL tenant_id = role global
permissions           (id, resource, action)                   -- global, no tenant
role_permissions      (role_id, permission_id)
user_tenant_roles     (user_id, tenant_id, role_id)
```

**Permisos tienen el formato `<resource>:<action>`:**

- `jobs:create`, `jobs:read`, `jobs:update`, `jobs:delete`, `jobs:publish`
- `candidates:create`, `candidates:read`, `candidates:update`, `candidates:delete`
- `applications:read`, `applications:update-status`
- `interviews:schedule`, `interviews:cancel`
- `users:invite`, `users:remove`, `users:assign-role`
- `tenant:settings:read`, `tenant:settings:update`

### 8.4 Roles por defecto por tenant

Cada tenant al crearse DEBE tener estos roles inicializados (seed):

| Rol | Permisos incluidos |
|---|---|
| `tenant_admin` | `*` (todos, excepto super-platform) |
| `recruiter` | `jobs:*`, `candidates:*`, `applications:*`, `interviews:*` |
| `hiring_manager` | `jobs:read`, `candidates:read`, `applications:read`, `applications:update-status`, `interviews:*` |
| `viewer` | `*:read` |

### 8.5 Verificación de permisos

- **MUST** usar el middleware `rbac(<permission>)` en cada ruta que requiere permisos.
- **MUST** colocar el middleware `rbac` DESPUÉS de `auth` y `tenantResolver`.
- **MUST** evaluar permisos contra `req.user.permissionsInTenant[tenantId]` (pre-computado al resolver el tenant).
- **SHOULD** cachear los permisos del usuario en Redis con TTL 5 minutos, invalidando en cambios.

**Ejemplo:**

```typescript
router.post(
  '/',
  auth(),
  tenantResolver(),
  rbac('jobs:create'),
  jobsController.create
);
```

### 8.6 Frontend: checks de permisos

- **MUST** tener helpers `hasPermission(permission)` y `<CanAccess permission="...">` en `lib/permissions.ts`.
- **MUST** ocultar UI de acciones que el usuario no puede ejecutar.
- **MUST NOT** confiar solo en checks de frontend — el backend DEBE validar también.

---

## 9. Capa de Dominio (DDD)

### 9.1 Principios generales

- **MUST** modelar el dominio en términos del negocio, no en términos técnicos. Nombres de clases, métodos y eventos DEBEN reflejar el vocabulario de reclutamiento.
- **MUST** mantener la capa de dominio completamente pura: sin Prisma, sin Express, sin librerías HTTP, sin logging directo.
- **MUST NOT** exponer setters públicos en Entities. Los cambios de estado SOLO ocurren mediante métodos con nombres de intención (`publish()`, `close()`, `archive()`), no `setStatus()`.

### 9.2 Value Objects

Un Value Object DEBE:

- Extender la clase base `ValueObject<T>`.
- Ser inmutable (`readonly` en propiedades).
- Implementar `equals(other)`.
- Validar sus invariantes en el constructor (lanzar `InvalidValueError` si inválido).
- Ser creado vía factory `static create(...)` que retorna `Result<T, DomainError>`.

**Base class:**

```typescript
// shared/domain/value-object.ts
export abstract class ValueObject<T extends Record<string, unknown>> {
  protected readonly props: Readonly<T>;

  protected constructor(props: T) {
    this.props = Object.freeze(props);
  }

  public equals(vo?: ValueObject<T>): boolean {
    if (vo === null || vo === undefined) return false;
    if (vo.props === undefined) return false;
    return JSON.stringify(this.props) === JSON.stringify(vo.props);
  }
}
```

### 9.3 Entities / Aggregates

Una Entity DEBE:

- Tener una identidad única (ID como Value Object).
- Extender `AggregateRoot<TId>` si es raíz de agregado.
- Encapsular invariantes del dominio (no permitir estados inválidos).
- Exponer métodos de comportamiento, no propiedades.
- Registrar domain events cuando cambia estado relevante (`this.addDomainEvent(new JobCreatedEvent(...))`).
- Ser creada vía factory `static create(...)` y reconstituida vía `static reconstitute(...)` desde persistencia.

**Regla clave:** cualquier cambio de estado que otros bounded contexts deban conocer DEBE generar un Domain Event.

### 9.4 Aggregate Boundaries

- **MUST** mantener los aggregates pequeños. Un aggregate es la unidad transaccional.
- **MUST NOT** modificar más de un aggregate en la misma transacción de BD. Si se necesita coordinación, usar Domain Events (eventual consistency).
- **MUST** referenciar otros aggregates SOLO por ID (no por referencia directa al objeto).

**Ejemplo correcto en `Application`:**

```typescript
class Application {
  private readonly candidateId: CandidateId;  // ✅ por ID
  private readonly jobId: JobId;              // ✅ por ID
  // NO: private readonly candidate: Candidate;  ❌
}
```

### 9.5 Domain Events

Un Domain Event DEBE:

- Ser inmutable.
- Tener nombre en pasado (`JobCreatedEvent`, no `CreateJob` ni `JobCreationEvent`).
- Incluir: `eventId`, `aggregateId`, `tenantId`, `occurredAt`, `eventVersion`, `payload`.
- Serializarse a JSON sin pérdida.

**Base class:**

```typescript
// shared/domain/domain-event.ts
export abstract class DomainEvent {
  public readonly eventId: string;
  public readonly occurredAt: Date;
  public readonly eventVersion: number = 1;

  constructor(
    public readonly aggregateId: string,
    public readonly tenantId: string,
    clock: Clock,
    idGenerator: IdGenerator,
  ) {
    this.eventId = idGenerator.generate();
    this.occurredAt = clock.now();
  }

  abstract get eventName(): string;
  abstract toPayload(): Record<string, unknown>;
}
```

### 9.6 Repositorios (puertos)

Repositorios son **interfaces en `domain/ports/`**, NO clases concretas.

- **MUST** definir solo métodos necesarios para la lógica de dominio (`findById`, `save`, `delete`). No exponer métodos genéricos tipo `executeQuery`.
- **MUST** tomar `tenantId` (o `TenantContext`) como parámetro.
- **MUST** retornar `Promise<Aggregate | null>` o `Promise<Aggregate>` (lanzar si no encuentra es decisión del caller).
- **MUST NOT** retornar modelos de Prisma ni entidades de BD. Siempre Aggregates del dominio.

### 9.7 Errores de dominio

- **MUST** crear un archivo `<module>.errors.ts` en `domain/errors/`.
- **MUST** extender de `DomainError` (base compartida).
- **MUST** tener un `code` único, `message` human-readable y opcionalmente `details`.

```typescript
// shared/domain/errors/domain-error.ts
export abstract class DomainError extends BaseError {
  abstract readonly code: string;
}

// modules/jobs/domain/errors/job.errors.ts
export class JobTitleTooShortError extends DomainError {
  readonly code = 'JOB_TITLE_TOO_SHORT';
  constructor(actualLength: number, minLength: number) {
    super(`Job title must be at least ${minLength} characters. Got ${actualLength}.`);
  }
}
```

### 9.8 Regla de oro

**El código en `domain/` DEBE poder ejecutarse sin Node.js instalado** (conceptualmente). Si un archivo en `domain/` necesita importar algo de `infrastructure/`, la arquitectura está mal.

---

## 10. Capa de Aplicación (Use Cases + CQRS ligero)

### 10.1 Commands vs Queries

- **Commands** modifican estado → viven en `application/commands/<verb>-<n>/`.
- **Queries** leen estado → viven en `application/queries/<verb>-<n>/`.

Cada command/query tiene su propio folder con tres archivos mínimo:

```
create-job/
├── create-job.command.ts     # DTO de entrada (clase simple con props)
├── create-job.handler.ts     # Lógica de orquestación
└── create-job.handler.spec.ts
```

### 10.2 Command Handler

Un Command Handler DEBE:

- Implementar la interface `CommandHandler<TCommand, TResult>`.
- Tener un único método público: `execute(context, command)`.
- Tomar dependencias por constructor (inyección).
- **NO contener lógica de dominio** — solo orquestar.
- Validar autorización de alto nivel (ej: tenant match, permiso RBAC ya verificado por middleware).
- Cargar aggregates del repositorio, invocar métodos de dominio, persistir.
- Publicar domain events tras commit exitoso.
- Retornar un DTO de salida tipado (no la entidad completa).

**Regla de longitud:** un handler DEBE caber en menos de 60 líneas. Si excede, extraer a un Domain Service.

### 10.3 Query Handler

Un Query Handler DEBE:

- Implementar `QueryHandler<TQuery, TResult>`.
- Retornar DTOs planos (no aggregates).
- **MAY** saltar el repositorio de dominio y consultar Prisma directamente para proyecciones optimizadas.
- **MUST** siempre filtrar por `tenantId`.
- **MUST NOT** modificar estado.
- **MUST NOT** publicar eventos.

### 10.4 DTOs de aplicación

- **MUST** definir DTOs de entrada (Command/Query) como clases simples con `readonly` properties.
- **MUST** definir DTOs de salida en `application/dtos/`.
- **MUST NOT** mezclar DTOs de aplicación con schemas Zod de la capa HTTP. Son cosas distintas.

### 10.5 Transacciones

- **MUST** usar `UnitOfWork` (puerto definido en `shared/domain/ports/unit-of-work.port.ts`) para envolver operaciones que tocan múltiples aggregates o involucran el Outbox.
- Implementación con Prisma usa `$transaction`.
- **MUST NOT** comenzar transacciones dentro de la capa de dominio.
- **MUST** publicar eventos SOLO después de commit exitoso (patrón Outbox, sección 11.4).

### 10.6 Inyección de dependencias

- **MUST** usar inyección manual por constructor. **NO usar** decoradores tipo NestJS ni librerías de DI (tsyringe, InversifyJS).
- Cada módulo tiene su `<module>.module.ts` que compone las dependencias:

```typescript
// modules/jobs/jobs.module.ts
export function createJobsModule(deps: {
  prisma: PrismaClient;
  clock: Clock;
  idGenerator: IdGenerator;
  eventBus: EventBus;
}): JobsModule {
  const jobRepository = new PrismaJobRepository(deps.prisma);
  const createJobHandler = new CreateJobHandler(
    jobRepository,
    deps.clock,
    deps.idGenerator,
    deps.eventBus,
  );
  // ...
  return { controllers: { jobsController: new JobsController(createJobHandler, ...) } };
}
```

---

## 11. Capa de Infraestructura

### 11.1 Prisma y el schema

- **MUST** mantener un único `schema.prisma` en `apps/api/prisma/`.
- **MUST** usar el generador `prisma-client-js` (no TypeScript de edge).
- **MUST** mapear nombres camelCase del modelo a snake_case de la BD con `@map()` y `@@map()`.
- **MUST** definir `@index` para `tenantId` y columnas frecuentemente filtradas.
- **MUST** usar `@@index([tenantId, ...otras])` con `tenantId` como primera columna.

**Ejemplo mínimo del schema:**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Tenant {
  id        String   @id @default(cuid())
  slug      String   @unique
  name      String
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt       @map("updated_at")

  jobs Job[]

  @@map("tenants")
}

model Job {
  id                 String    @id @default(cuid())
  tenantId           String    @map("tenant_id")
  title              String
  department         String
  location           String
  workMode           String    @map("work_mode")      // remote | hybrid | onsite
  description        String    @db.Text
  status             String                            // draft | open | closed | archived
  createdBy          String    @map("created_by")
  createdAt          DateTime  @default(now())         @map("created_at")
  updatedAt          DateTime  @updatedAt              @map("updated_at")
  publishedAt        DateTime? @map("published_at")
  closedAt           DateTime? @map("closed_at")

  tenant Tenant @relation(fields: [tenantId], references: [id])

  @@index([tenantId, status], map: "idx_jobs_tenant_id_status")
  @@index([tenantId, createdAt], map: "idx_jobs_tenant_id_created_at")
  @@map("jobs")
}
```

### 11.2 Migraciones

- **MUST** usar `prisma migrate dev` en desarrollo y `prisma migrate deploy` en producción.
- **MUST** commitear los archivos de migración generados.
- **MUST NOT** editar migraciones ya aplicadas en producción. Crear una nueva migración correctiva.
- **MUST** probar migraciones en ambiente de staging antes de producción.
- **MUST** nombrar migraciones con formato: `YYYYMMDDHHMMSS_<snake_case_description>`.
- **MUST** incluir en cada PR que tenga migración:
  - Nota en la descripción del PR explicando el cambio.
  - Verificación de backward compatibility con la versión anterior del código (para deploys sin downtime).

### 11.3 Repositorios concretos

- **MUST** implementar la interface de `domain/ports/<n>.repository.ts`.
- **MUST** usar un mapper (`<n>.mapper.ts`) para convertir entre modelo Prisma y Aggregate de dominio.
- **MUST NOT** filtrar resultados sin `tenantId`.

**Estructura del mapper:**

```typescript
// modules/jobs/infrastructure/persistence/job.mapper.ts
export class JobMapper {
  static toDomain(raw: PrismaJob): Job {
    return Job.reconstitute({
      id: JobId.create(raw.id).getValue(),
      tenantId: raw.tenantId,
      title: JobTitle.create(raw.title).getValue(),
      // ...
    });
  }

  static toPersistence(job: Job): Omit<PrismaJob, 'tenant'> {
    const props = job.toPrimitives();
    return {
      id: props.id,
      tenantId: props.tenantId,
      title: props.title,
      // ...
    };
  }
}
```

### 11.4 Outbox Pattern para Domain Events

Para garantizar que los eventos se publiquen exactamente cuando el aggregate se persiste (sin perder eventos ante fallos):

**Tabla:**

```prisma
model OutboxEvent {
  id           String   @id @default(cuid())
  tenantId    String   @map("tenant_id")
  aggregateId  String   @map("aggregate_id")
  eventName    String   @map("event_name")
  eventVersion Int      @map("event_version")
  payload      Json
  occurredAt   DateTime @map("occurred_at")
  publishedAt  DateTime? @map("published_at")
  attempts     Int      @default(0)
  lastError    String?  @map("last_error")

  @@index([publishedAt, occurredAt], map: "idx_outbox_pending")
  @@index([tenantId], map: "idx_outbox_tenant_id")
  @@map("outbox_events")
}
```

**Flujo:**

1. El Command Handler abre una transacción Prisma.
2. Persiste el aggregate modificado.
3. Persiste los domain events como filas en `outbox_events` dentro de la misma transacción.
4. Commit.
5. Un worker (`outbox-publisher`) poll-ea cada 1 segundo, toma filas con `publishedAt IS NULL`, las publica a BullMQ, y las marca como publicadas.
6. Reintentos con backoff exponencial ante fallos.

**Reglas:**

- **MUST** usar Outbox para todos los domain events que crucen bounded contexts.
- **MAY** publicar directamente in-process para eventos estrictamente dentro del mismo módulo.
- **MUST** hacer el worker idempotente (el mismo evento puede publicarse más de una vez).
- **MUST** los event handlers deben ser idempotentes.

### 11.5 Redis

- **MUST** usar Redis para: cache de permisos RBAC, cache de tenant-by-slug, BullMQ queues, rate limiting.
- **MUST NOT** usar Redis como source of truth.
- **MUST** fijar TTL en TODA clave. Sin excepciones.
- **MUST** usar key prefixing: `<env>:<feature>:<key>` (ej: `prod:tenant-by-slug:acme`).

### 11.6 BullMQ

- **MUST** definir colas en `shared/infrastructure/bullmq/queues.ts`.
- **MUST** tener un Processor por tipo de job en `workers/processors/`.
- **MUST** configurar retry con backoff exponencial (3 intentos, base 1s).
- **MUST** configurar Dead Letter Queue para jobs fallidos.
- **MUST** monitorear colas con BullMQ Board expuesto solo a platform admins.

**Colas iniciales:**

- `domain-events` (publicación de eventos del Outbox).
- `emails` (envío de emails).
- `notifications` (notificaciones in-app).
- `reports` (generación de reportes pesados).

### 11.7 Configuración de ambiente

- **MUST** validar variables de entorno con Zod al arrancar. Si falla, el proceso DEBE crashear inmediatamente.
- **MUST** NO acceder a `process.env` fuera del módulo de config.
- **MUST** listar TODAS las variables en `.env.example`.

```typescript
// apps/api/src/config/env.schema.ts
import { z } from 'zod';

export const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'staging', 'production']),
  PORT: z.coerce.number().int().positive().default(4000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  INTERNAL_JWT_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  SENTRY_DSN: z.string().url().optional(),
  OTEL_EXPORTER_OTLP_ENDPOINT: z.string().url().optional(),
  LOG_LEVEL: z.enum(['trace', 'debug', 'info', 'warn', 'error']).default('info'),
});

export type Env = z.infer<typeof envSchema>;
```

---

## 12. API REST

### 12.1 Versionado

- **MUST** versionar en el path: `/api/v1/...`.
- **MUST NOT** romper compatibilidad en una versión ya publicada. Para cambios breaking, crear `/api/v2/`.
- **MUST** mantener v1 funcionando durante un periodo de deprecación mínimo de 6 meses tras lanzar v2.
- **MUST** agregar header `Sunset` y `Deprecation` en endpoints deprecados.

### 12.2 Estructura de rutas

Patrón base: `/api/v1/t/:tenantSlug/<resource>`

| Operación | Método | Path | Status esperado |
|---|---|---|---|
| Listar | `GET` | `/jobs` | `200` |
| Obtener por ID | `GET` | `/jobs/:id` | `200` / `404` |
| Crear | `POST` | `/jobs` | `201` + `Location` header |
| Actualizar parcial | `PATCH` | `/jobs/:id` | `200` |
| Reemplazar | `PUT` | `/jobs/:id` | `200` |
| Eliminar | `DELETE` | `/jobs/:id` | `204` |
| Acción específica | `POST` | `/jobs/:id/publish` | `200` |

### 12.3 Formato de request/response

**Request:**

- **MUST** usar `Content-Type: application/json`.
- **MUST** validar body con Zod. Si falla → `400 VALIDATION_ERROR`.
- **MUST** aceptar `camelCase` en JSON.

**Response exitosa:**

```json
{
  "data": { ... },
  "meta": {
    "requestId": "req_abc123"
  }
}
```

**Response de lista paginada:**

```json
{
  "data": [ ... ],
  "meta": {
    "requestId": "req_abc123",
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "totalItems": 147,
      "totalPages": 8
    }
  }
}
```

**Response de error:**

```json
{
  "error": {
    "code": "JOB_TITLE_TOO_SHORT",
    "message": "Job title must be at least 3 characters.",
    "details": { "actualLength": 2, "minLength": 3 }
  },
  "meta": {
    "requestId": "req_abc123"
  }
}
```

### 12.4 Paginación

- **MUST** soportar paginación offset-based por defecto: `?page=1&pageSize=20`.
- **MUST** limitar `pageSize` a máximo 100.
- **MUST** tener `pageSize` default de 20.
- **SHOULD** soportar cursor-based (`?cursor=...&limit=20`) para listas grandes (>10k registros).
- **MUST** retornar siempre `totalItems` y `totalPages` en responses de lista.

### 12.5 Filtros y ordenamiento

- **MUST** usar query params: `?status=open&sort=-createdAt&q=developer`.
- **MUST** documentar en Zod schema qué filtros soporta cada endpoint.
- **MUST NOT** exponer filtros arbitrarios tipo `?where={"foo":"bar"}`.
- **MUST** prefijo `-` para orden descendente: `?sort=-createdAt`.

### 12.6 Status codes permitidos

- `200 OK` — éxito con body.
- `201 Created` — recurso creado. DEBE incluir header `Location`.
- `204 No Content` — éxito sin body.
- `400 Bad Request` — error de validación de entrada.
- `401 Unauthorized` — no autenticado.
- `403 Forbidden` — autenticado pero sin permiso.
- `404 Not Found` — recurso no existe.
- `409 Conflict` — conflicto de estado (ej: email duplicado).
- `422 Unprocessable Entity` — body sintácticamente válido pero semánticamente inválido (regla de dominio violada).
- `429 Too Many Requests` — rate limit.
- `500 Internal Server Error` — error inesperado del servidor.

**MUST NOT** usar otros status codes sin justificación documentada.

### 12.7 Rate limiting

- **MUST** aplicar rate limit por `userId + tenantId`.
- **MUST** configurar por endpoint: endpoints de escritura más restrictivos.
- **MUST** retornar headers `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.

### 12.8 Idempotencia

- **MUST** soportar header `Idempotency-Key` en endpoints `POST` críticos (creación de vacantes, candidatos).
- **MUST** almacenar la key + response por 24 horas en Redis.
- **MUST** retornar la misma response si llega la misma key.

---

## 13. Validación y Manejo de Errores

### 13.1 Jerarquía de errores

```
BaseError (shared/domain/errors/base-error.ts)
├── DomainError             (errores de reglas de negocio)
│   ├── InvalidValueError   (para Value Objects con datos inválidos)
│   ├── NotFoundError       (aggregate no encontrado)
│   ├── ConflictError       (estado inconsistente)
│   └── <module>Error       (específicos por módulo)
├── ApplicationError        (errores de la capa de aplicación)
│   ├── UnauthorizedError
│   ├── ForbiddenError
│   └── TenantMismatchError
├── ValidationError         (errores de validación de input — Zod)
└── InfrastructureError     (errores de BD, red, etc.)
    ├── DatabaseError
    ├── ExternalServiceError
    └── TimeoutError
```

### 13.2 Definición de BaseError

```typescript
export abstract class BaseError extends Error {
  abstract readonly code: string;
  public readonly details?: Record<string, unknown>;

  constructor(message: string, details?: Record<string, unknown>) {
    super(message);
    this.name = this.constructor.name;
    this.details = details;
    Error.captureStackTrace?.(this, this.constructor);
  }

  toJSON() {
    return {
      code: this.code,
      message: this.message,
      details: this.details,
    };
  }
}
```

### 13.3 Mapeo de errores a HTTP

En el middleware global de error handling (`shared/interface/middleware/error-handler.ts`):

| Error class | Status HTTP |
|---|---|
| `ValidationError` | 400 |
| `UnauthorizedError` | 401 |
| `ForbiddenError`, `TenantMismatchError` | 403 |
| `NotFoundError` | 404 |
| `ConflictError` | 409 |
| `DomainError` (otros) | 422 |
| `InfrastructureError`, cualquier otro | 500 |

### 13.4 Validación con Zod

- **MUST** tener schemas Zod en `packages/contracts/` para compartir entre frontend y backend.
- **MUST** validar en el controller antes de llamar al handler.
- **MUST** usar `safeParse` y convertir errores de Zod a `ValidationError`.

```typescript
// packages/contracts/src/jobs/create-job.schema.ts
import { z } from 'zod';

export const createJobSchema = z.object({
  title: z.string().trim().min(3).max(120),
  department: z.string().trim().min(2).max(80),
  location: z.string().trim().min(2).max(120),
  workMode: z.enum(['remote', 'hybrid', 'onsite']),
  description: z.string().trim().min(20).max(10_000),
  status: z.enum(['draft', 'open']).default('draft'),
});

export type CreateJobInput = z.infer<typeof createJobSchema>;
```

### 13.5 Política de logging de errores

- **MUST** loguear errores 5xx siempre con stack trace y contexto completo.
- **MUST** loguear 4xx como `warn` (no `error`), sin stack trace.
- **MUST** enviar 5xx a Sentry.
- **MUST NOT** enviar 4xx a Sentry (ruido).
- **MUST** redactar PII y secretos antes de loguear.

### 13.6 Respuestas de error al cliente

- **MUST NOT** exponer stack traces en producción.
- **MUST** incluir `requestId` en cada respuesta de error para correlación.
- **MAY** incluir `details` con información específica del error (ej: campos que fallaron validación).

---

## 14. Frontend (Next.js)

### 14.1 Principios generales

- **MUST** usar App Router exclusivamente. Pages Router **prohibido**.
- **MUST** preferir Server Components por defecto. Usar `'use client'` solo cuando sea estrictamente necesario (interactividad, hooks de cliente).
- **MUST** usar Server Actions para mutations. **NO crear** API routes de Next.js para lógica de negocio.
- **MUST** consumir el backend Express vía el cliente HTTP (`lib/api-client.ts`) desde Server Components/Actions, nunca desde Client Components directamente.
- **MUST NOT** exponer el backend Express directamente al navegador. Todo pasa por Next.js (SSR proxy pattern).

### 14.2 Arquitectura feature-based

Todo código funcional vive bajo `src/features/<feature>/`:

```
features/jobs/
├── components/       # Componentes de UI específicos del feature
├── hooks/            # TanStack Query hooks (use-jobs, use-create-job)
├── actions/          # Server Actions
├── schemas/          # Re-export de @ats/contracts (o extensiones)
└── types/            # Tipos auxiliares del feature
```

- **MUST NOT** mezclar código de features distintos. `jobs/` no importa de `candidates/` directamente — si necesitan comunicarse, usar contratos compartidos.
- **MUST** mantener componentes genéricos en `src/components/`.

### 14.3 Shadcn/ui y componentes UI

- **MUST** usar shadcn/ui como sistema de componentes base. NO agregar librerías de componentes externas.
- **MUST** generar componentes shadcn en `src/components/ui/` mediante el CLI.
- **MUST NOT** editar los archivos generados por shadcn sin documentar el motivo (los regenera el agente si se solicita).
- **MUST** componer componentes del dominio a partir de los de `ui/`. Ejemplo: un `JobCard` usa `Card`, `Badge`, `Button` de shadcn.
- **MUST** usar Tailwind CSS. NO CSS-in-JS. NO CSS modules.
- **MUST** usar el helper `cn()` de `lib/utils.ts` para concatenar classes condicionales.
- **MUST** respetar los design tokens definidos en `tailwind.config.ts` (colores, spacing, radius, fonts). NO hardcodear colores hex en componentes.

### 14.4 Routing con tenants

Estructura de rutas:

```
/                              → landing pública
/login                         → auth
/t/[tenantSlug]                → dashboard del tenant
/t/[tenantSlug]/jobs           → lista de vacantes
/t/[tenantSlug]/jobs/new       → crear vacante
/t/[tenantSlug]/jobs/[jobId]   → detalle de vacante
```

- **MUST** validar en el `layout.tsx` de `/t/[tenantSlug]/` que el usuario tiene acceso al tenant. Si no → `redirect('/login')` o `notFound()`.
- **MUST** resolver el `tenantId` del slug y guardarlo en contexto (React Context en Server Component → pasado a Client Components vía props).

### 14.5 Server Actions

- **MUST** vivir en `features/<feature>/actions/`.
- **MUST** usar directiva `'use server'` al inicio del archivo.
- **MUST** validar input con el Zod schema compartido.
- **MUST** obtener sesión con `auth()` y verificar permisos antes de llamar al backend.
- **MUST** llamar al backend Express vía `apiClient` (no hacer lógica de negocio en la action).
- **MUST** retornar un tipo `ActionResult<T>` estandarizado.

```typescript
// lib/action-result.ts
export type ActionResult<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; code: string; message: string; fieldErrors?: Record<string, string[]> };
```

### 14.6 Data fetching

**Server Components:**

- **MUST** hacer fetches directamente con `async` en el componente.
- **MUST** pasar datos a Client Components como props.
- **MUST** usar `cache()` de React cuando sea apropiado.
- **SHOULD** pre-poblar TanStack Query con `dehydrate` para hydratación client-side cuando haya interactividad posterior.

**Client Components:**

- **MUST** usar TanStack Query para todo server state.
- **MUST** organizar query keys: `['tenant', tenantId, 'feature', ...params]`.
- **MUST** invalidar queries relacionadas tras mutations exitosas.
- **MUST NOT** usar `useEffect` + `fetch` para data fetching. Siempre TanStack Query.

### 14.7 Zustand stores

- **MUST** crear un store por dominio de UI, no un store global god-object.
- **MUST** usar la pattern con slices si un store crece > 150 líneas.
- **MUST** usar `persist` middleware solo para UI preferences (theme, sidebar collapsed). NO persistir datos de servidor.
- **MUST NOT** poner server state en Zustand. Eso va en TanStack Query.

**Stores iniciales:**

- `ui.store.ts` → sidebar open/closed, theme, modal stack.
- `tenant.store.ts` → tenant activo actual (persisted).

### 14.8 Formularios

- **MUST** usar `react-hook-form` con el resolver `@hookform/resolvers/zod`.
- **MUST** usar el schema Zod compartido de `@ats/contracts`.
- **MUST** mostrar errores inline por campo.
- **MUST** deshabilitar submit mientras `isSubmitting`.
- **MUST** mostrar loading states claros.

### 14.9 Accesibilidad

- **MUST** usar los componentes de shadcn/ui (que están construidos sobre Radix, accesibles por defecto).
- **MUST** tener `alt` en imágenes.
- **MUST** tener label asociado a cada input.
- **MUST** manejar estados de focus y navegación por teclado.
- **MUST** respetar `prefers-reduced-motion` para animaciones.

### 14.10 Rendimiento

- **MUST** usar `next/image` para imágenes.
- **MUST** usar `next/font` para fuentes.
- **MUST** lazy-load componentes pesados con `dynamic()`.
- **SHOULD** mantener el JS de cliente inicial bajo 100 KB gzipped en páginas críticas.

---

## 15. Testing

### 15.1 Pirámide de testing

| Tipo | Framework | Cantidad relativa | Velocidad esperada |
|---|---|---|---|
| Unit | Jest | 70% | < 50ms/test |
| Integration | Jest + testcontainers | 20% | < 2s/test |
| E2E | Playwright | 10% | < 30s/test |

### 15.2 Cobertura mínima

- **Domain layer:** 95% de cobertura de líneas y branches.
- **Application layer:** 90%.
- **Infrastructure:** 70% (integraciones más pesadas).
- **Interface (controllers):** 80%.
- **Frontend:** 70% en utils/hooks, tests de comportamiento para componentes críticos.

**MUST** fallar CI si cobertura cae bajo el umbral.

### 15.3 Unit tests

- **MUST** vivir junto al archivo testeado (`create-job.handler.spec.ts` junto a `create-job.handler.ts`).
- **MUST** mockear TODAS las dependencias (repositorios, clock, event bus).
- **MUST NOT** tocar BD real en unit tests.
- **MUST** seguir la estructura `arrange / act / assert` con comentarios o bloques visibles.
- **MUST** testar tanto happy path como casos de error.
- **MUST** testar los invariants de cada Value Object y Entity.

**Convenciones de naming:**

```typescript
describe('CreateJobHandler', () => {
  describe('when the command is valid', () => {
    it('should create the job and publish a JobCreatedEvent', async () => { ... });
  });

  describe('when the title is too short', () => {
    it('should throw JobTitleTooShortError', async () => { ... });
  });
});
```

### 15.4 Integration tests

- **MUST** vivir en `modules/<m>/__tests__/integration/`.
- **MUST** usar testcontainers para levantar PostgreSQL y Redis reales.
- **MUST** usar migraciones reales en setup.
- **MUST** limpiar datos entre tests (truncate, no drop).
- **MUST** testear el happy path completo del módulo (controller → handler → repo → BD).
- **MUST** testear el comportamiento multi-tenant: operación con `tenant A` NO debe ver datos de `tenant B`.

### 15.5 E2E tests (Playwright)

- **MUST** vivir en `apps/web/e2e/specs/`.
- **MUST** usar un tenant de test dedicado sembrado antes de correr.
- **MUST** usar `data-testid` para selectores estables. NO seleccionar por texto traducible ni clases CSS.
- **MUST** cubrir los flujos críticos: login, crear tenant, crear vacante, crear candidato, aplicar candidato a vacante, agendar entrevista.
- **MUST** correr e2e en CI contra un stack Docker efímero.

### 15.6 Fixtures y factories

- **MUST** crear `test/fixtures/<aggregate>.fixture.ts` con factory functions:

```typescript
// test/fixtures/job.fixture.ts
export function makeJob(overrides?: Partial<JobProps>): Job {
  return Job.create({
    id: 'job-test-id',
    tenantId: 'tenant-test-id',
    title: 'Senior Backend Engineer',
    // ...
    ...overrides,
  }).getValue();
}
```

### 15.7 Test doubles

- **MUST** preferir fakes (implementaciones reales en memoria) sobre mocks cuando sea posible.
- **MAY** usar mocks de Jest solo para verificar interacciones (ej: que se publicó un evento).

**Ejemplo de fake:**

```typescript
// test/fakes/in-memory-job.repository.ts
export class InMemoryJobRepository implements JobRepository {
  private jobs = new Map<string, Job>();

  async findById(tenantId: string, id: JobId): Promise<Job | null> {
    const job = this.jobs.get(id.value);
    if (!job || job.tenantId !== tenantId) return null;
    return job;
  }

  async save(job: Job): Promise<void> {
    this.jobs.set(job.id.value, job);
  }
}
```

### 15.8 CI

- **MUST** correr en cada PR: lint + typecheck + unit tests + integration tests.
- **MUST** correr e2e al menos una vez antes de merge a main.
- **MUST** correr `pnpm audit` semanalmente como job programado.

---

## 16. Logging, Auditoría y Observabilidad

### 16.1 Logging estructurado (Pino)

- **MUST** usar Pino como único logger.
- **MUST** producir logs JSON estructurados en todos los ambientes (incluso dev, con `pino-pretty` solo para lectura humana en terminal).
- **MUST NOT** usar `console.log/error/warn` en código de producción. Solo el logger.

**Campos obligatorios en cada log:**

- `level`
- `time`
- `msg`
- `requestId` (si aplica)
- `tenantId` (si aplica)
- `userId` (si aplica)
- `service` (ej: `api`, `web`, `worker-emails`)

**Ejemplo de logger configurado:**

```typescript
// packages/logger/src/index.ts
import pino from 'pino';

export function createLogger(service: string) {
  return pino({
    level: process.env.LOG_LEVEL ?? 'info',
    base: { service },
    timestamp: pino.stdTimeFunctions.isoTime,
    redact: {
      paths: ['req.headers.authorization', 'req.headers.cookie', '*.password', '*.token'],
      censor: '[REDACTED]',
    },
  });
}
```

### 16.2 Niveles de log

| Nivel | Cuándo usar |
|---|---|
| `trace` | Detalle muy granular, solo dev |
| `debug` | Información para debugging |
| `info` | Eventos relevantes del flujo normal (requests, eventos publicados) |
| `warn` | Condiciones inesperadas recuperables (4xx, retry, fallback) |
| `error` | Errores que requieren atención (5xx, fallos de integración) |
| `fatal` | Errores que harán crashear el proceso |

### 16.3 Correlation ID

- **MUST** generar un `requestId` (formato `req_<nanoid>`) en el primer middleware.
- **MUST** propagar el `requestId` en:
  - Logs.
  - Headers HTTP salientes a servicios internos.
  - Metadata de jobs BullMQ.
  - Spans de OpenTelemetry.
- **MUST** retornar el `requestId` en `meta.requestId` de toda response.

### 16.4 Auditoría

**Eventos que DEBEN auditarse (persistir en `audit_logs`):**

- Creación, modificación, eliminación de Jobs, Candidates, Applications.
- Cambios de estado de Applications (ej: `shortlisted` → `rejected`).
- Agendar/cancelar Interviews.
- Invitación, remoción, cambio de rol de Users.
- Login exitoso, login fallido, logout.
- Cambios en configuración del Tenant.
- Acceso a CVs descargados.

**Esquema de `audit_logs`:**

```prisma
model AuditLog {
  id           String   @id @default(cuid())
  tenantId     String   @map("tenant_id")
  userId       String?  @map("user_id")      // puede ser null si es acción del sistema
  action       String                         // ej: "jobs.created"
  resourceType String   @map("resource_type") // ej: "job"
  resourceId   String   @map("resource_id")
  before       Json?
  after        Json?
  ipAddress    String?  @map("ip_address")
  userAgent    String?  @map("user_agent")
  requestId    String   @map("request_id")
  occurredAt   DateTime @default(now()) @map("occurred_at")

  @@index([tenantId, occurredAt], map: "idx_audit_tenant_time")
  @@index([tenantId, resourceType, resourceId], map: "idx_audit_resource")
  @@map("audit_logs")
}
```

### 16.5 Tracing (OpenTelemetry)

- **MUST** instrumentar Express con `@opentelemetry/instrumentation-express`.
- **MUST** instrumentar Prisma con `@opentelemetry/instrumentation-pg`.
- **MUST** exportar traces a un collector (CloudWatch X-Ray en AWS, o Tempo/Jaeger).
- **MUST** crear spans custom para commands/queries importantes:
  ```typescript
  await tracer.startActiveSpan('CreateJobHandler.execute', async (span) => {
    span.setAttribute('tenant.id', context.tenantId);
    // ... lógica
    span.end();
  });
  ```

### 16.6 Métricas

- **MUST** exportar métricas básicas: requests por ruta, latencias (p50/p95/p99), error rate.
- **MUST** exportar métricas de negocio: jobs creados/día/tenant, candidates creados/día/tenant, interviews agendadas.
- **MUST** usar OpenTelemetry Metrics API.

### 16.7 Sentry

- **MUST** capturar errores 5xx automáticamente.
- **MUST** incluir contexto: `tenantId`, `userId`, `requestId`.
- **MUST NOT** enviar PII (email, nombre completo) a Sentry. Solo IDs.
- **MUST** tener alertas configuradas por proyecto con umbrales: nuevo issue, spike de errors.

### 16.8 PII y cumplimiento

- **MUST** tratar los siguientes campos como PII y NO loguear:
  - Emails completos (loguear solo dominios o IDs hash).
  - Nombres completos.
  - Teléfonos.
  - CVs.
  - Documentos de identidad.
- **MUST** cifrar en reposo campos sensibles en BD usando `pgcrypto` o columna cifrada a nivel aplicación.
- **MUST** soportar borrado de datos por solicitud (GDPR "derecho al olvido") — anonimizar, no eliminar físicamente para preservar auditoría.

---

## 17. Git Workflow

### 17.1 Branching

- **MUST** usar trunk-based development con ramas cortas (< 3 días).
- **MUST** nombrar ramas:
  - `feat/<ticket-id>-<short-desc>` — nueva feature.
  - `fix/<ticket-id>-<short-desc>` — bugfix.
  - `refactor/<ticket-id>-<short-desc>` — refactor sin cambio funcional.
  - `chore/<short-desc>` — tareas de mantenimiento sin ticket.
  - `docs/<short-desc>` — solo documentación.
- **MUST** ramificar desde `main`.
- **MUST** rebasear sobre `main` antes de abrir PR (no merge commits).

### 17.2 Commits

- **MUST** seguir Conventional Commits: `<type>(<scope>): <subject>`.
- **Types permitidos:** `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`, `build`, `ci`.
- **Scope:** nombre del módulo afectado (`jobs`, `candidates`, `auth`, `web`, `api`).
- **Subject:** en imperativo, minúscula inicial, sin punto final, < 72 caracteres.
- **MUST** mantener commits atómicos (un commit = un cambio lógico).

Ejemplos correctos:
- `feat(jobs): add job publish endpoint`
- `fix(auth): redirect to correct tenant after login`
- `refactor(candidates): extract name normalization to value object`

### 17.3 Pull Requests

- **MUST** tener descripción con:
  - Qué cambia (1-3 frases).
  - Por qué cambia (contexto de negocio / ticket).
  - Cómo probarlo (pasos manuales si aplica).
  - Screenshots para cambios de UI.
  - Checklist de la sección 18.
- **MUST** tener al menos un reviewer.
- **MUST** pasar todos los checks de CI.
- **MUST NOT** mergear con tests rojos, lint fallando, o coverage bajo el mínimo.
- **SHOULD** tener < 400 líneas cambiadas. PRs más grandes deben dividirse.

### 17.4 Protección de ramas

- **MUST** proteger `main`: requerir PR, review aprobado, CI verde, branch up-to-date.
- **MUST NOT** permitir force push a `main`.
- **MUST** usar squash-merge para PRs (mantiene historia limpia en main).

---

## 18. Checklist del Agente

El Agente de IA **DEBE** ejecutar este checklist antes de considerar cualquier tarea como completa.

### 18.1 Checklist pre-ejecución (antes de escribir código)

- [ ] Identifiqué el bounded context al que pertenece la tarea.
- [ ] Verifiqué que la tarea no requiere agregar dependencias no listadas en la sección 3.
- [ ] Localicé la sección de este documento que aplica.
- [ ] Identifiqué las reglas MUST que aplican.
- [ ] Verifiqué que el caso de uso ya existe (para modificaciones) o que hay que crearlo (para nuevas features).
- [ ] Verifiqué si hay un template relevante en la sección 20.

### 18.2 Checklist por cada archivo creado/modificado

- [ ] Nombre del archivo cumple la convención de la sección 6.1.
- [ ] Ubicación del archivo es la correcta según la estructura de la sección 5.
- [ ] TypeScript estricto sin `any`, sin `@ts-ignore` sin justificación.
- [ ] Imports ordenados: externos → packages → módulos propios (agrupados por capa).
- [ ] Sin importaciones cruzadas entre módulos (solo vía `index.ts` público).
- [ ] Sin dependencias de capas externas en `domain/`.
- [ ] Sin acceso directo a `process.env` fuera de `config/`.
- [ ] Sin `console.log` — usar logger.
- [ ] Sin `Date.now()` / `new Date()` en domain/application — usar `Clock`.
- [ ] Sin `Math.random()` / `crypto.randomUUID()` en domain/application — usar `IdGenerator`.

### 18.3 Checklist para un Command/Query Handler

- [ ] Implementa la interface correspondiente.
- [ ] Recibe `TenantContext` como primer parámetro.
- [ ] Todas las dependencias inyectadas por constructor.
- [ ] Usa el repositorio del dominio, no Prisma directamente (excepto queries optimizadas).
- [ ] Filtra por `tenantId` en toda operación.
- [ ] Publica domain events cuando muta estado relevante.
- [ ] Tiene su test unitario (`.spec.ts`) junto al handler.
- [ ] El test cubre happy path + al menos un error path.

### 18.4 Checklist para un endpoint HTTP

- [ ] Ruta sigue el patrón `/api/v1/t/:tenantSlug/<resource>`.
- [ ] Middlewares aplicados en orden: `auth` → `tenantResolver` → `rbac(<permission>)` → controller.
- [ ] Schema Zod definido en `packages/contracts/`.
- [ ] Request validado con `safeParse` y convertido a `ValidationError` en fallo.
- [ ] Response sigue el formato estándar de la sección 12.3.
- [ ] Status codes de la sección 12.6.
- [ ] Test de integración cubre: happy path, validación fallida, permiso faltante, tenant mismatch.

### 18.5 Checklist para un componente React

- [ ] Server Component por defecto. `'use client'` solo si necesario.
- [ ] No importa código de backend Express directamente.
- [ ] Usa shadcn/ui + Tailwind, sin CSS custom innecesario.
- [ ] Tiene `data-testid` en elementos interactivos críticos.
- [ ] Props tipadas explícitamente.
- [ ] Usa hooks de TanStack Query para server state.
- [ ] Usa Zustand para client state global si aplica.
- [ ] Accesibilidad: labels, alt, keyboard navigation.
- [ ] Estados de loading, error y vacío manejados explícitamente.

### 18.6 Checklist de tests

- [ ] Unit tests junto al archivo.
- [ ] Integration tests en `__tests__/integration/` del módulo.
- [ ] Coverage no baja respecto al mínimo de la sección 15.2.
- [ ] Nombres descriptivos en `describe` / `it`.
- [ ] Sin tests flaky (determinísticos, sin `Math.random`, sin sleeps).

### 18.7 Checklist antes de abrir PR

- [ ] `pnpm lint` sin errores.
- [ ] `pnpm typecheck` sin errores.
- [ ] `pnpm test` todos los tests verdes.
- [ ] `pnpm test:integration` verde.
- [ ] Si toca migraciones: corrí `pnpm prisma migrate deploy` en una BD limpia y verifiqué que aplica bien.
- [ ] Actualicé el changelog si cambia algo observable.
- [ ] Actualicé este SDD si el cambio altera una regla.

---

## 19. Anti-Patrones Prohibidos

Esta sección lista **explícitamente** lo que el agente **NO DEBE** hacer. La lista crece con el tiempo; el agente DEBE sugerir nuevas entradas si detecta un mal patrón recurrente.

### 19.1 Arquitectura

- ❌ Importar Prisma en `domain/` o `application/`.
- ❌ Importar Express en `domain/` o `application/`.
- ❌ Usar decoradores de dependency injection (NestJS-style). Usar inyección manual por constructor.
- ❌ Tener una capa `services/` genérica que mezcla dominio, aplicación e infraestructura.
- ❌ Crear clases "God Object" que manejan múltiples aggregates.
- ❌ Un módulo importa directamente desde `modules/<otro>/domain/`.
- ❌ Modificar dos aggregates en la misma transacción de BD.

### 19.2 Código

- ❌ `any` sin justificación con comentario.
- ❌ `// eslint-disable` sin comentario explicando.
- ❌ Funciones de más de 50 líneas. Si ocurre, extraer.
- ❌ Clases de más de 300 líneas. Si ocurre, separar responsabilidades.
- ❌ Métodos con más de 5 parámetros. Usar objetos de parámetros.
- ❌ Ciclos `for` clásicos cuando hay alternativas funcionales (`.map`, `.filter`) y el dataset es acotado.
- ❌ Mutar arrays o objetos recibidos como parámetros.
- ❌ Comentarios comentando código muerto. Borrarlo.

### 19.3 Seguridad

- ❌ Concatenar strings en queries SQL.
- ❌ Confiar en `tenantId` del body o query params del cliente.
- ❌ Loguear headers completos de request (contienen `Authorization`).
- ❌ Loguear CVs, emails completos o datos personales de candidatos.
- ❌ Retornar mensajes de error detallados en producción que expongan estructura interna.
- ❌ Endpoints `DELETE` sin verificación de permiso explícita.

### 19.4 Base de datos

- ❌ Queries sin `tenantId` en el `where`.
- ❌ Índices ausentes en columnas de filtro frecuente.
- ❌ `SELECT *` implícito sin necesidad (usar `select` de Prisma para limitar campos).
- ❌ N+1 queries. Usar `include` o `in` batched.
- ❌ Migraciones destructivas (DROP COLUMN) sin plan de rollback.
- ❌ Editar una migración ya aplicada en producción.

### 19.5 Frontend

- ❌ `useEffect` + `fetch` para data fetching. Usar TanStack Query.
- ❌ Prop drilling profundo (>3 niveles). Usar composición, context puntual o Zustand.
- ❌ Mezclar server state con client state en el mismo store.
- ❌ Llamar al backend Express directamente desde Client Components.
- ❌ Hardcodear colores hex en componentes.
- ❌ Uso de `any` en props de componentes.
- ❌ Persistir datos de usuario sensibles en `localStorage`.

### 19.6 Testing

- ❌ Tests que dependen del orden de ejecución.
- ❌ Tests que usan la BD de desarrollo compartida.
- ❌ Tests con `setTimeout` reales esperando eventos.
- ❌ Mocks que validan lógica interna no observable.
- ❌ Tests sin asserts (que solo corren código).
- ❌ Snapshots grandes y frágiles como única assertion.

---

## 20. Templates Copy-Paste: Caso de Uso Completo "Crear Vacante"

Esta sección implementa **end-to-end** el caso de uso "Crear Vacante" descrito al inicio del proyecto:

> **Actor(es):** Recruiter, Admin
> **Descripción:** Permite registrar una nueva vacante con la información mínima operativa: título, área, ubicación, modalidad, descripción y estado. Este caso de uso inicia formalmente un proceso de reclutamiento dentro del sistema.
> **Precondiciones:** El usuario debe estar autenticado y contar con permisos de creación de vacantes.
> **Resultado esperado:** La vacante queda almacenada en estado borrador o abierta, lista para recibir postulaciones.
> **Prioridad:** Alta

El agente de IA **DEBE** usar estos templates como referencia exacta para cualquier otro caso de uso. Solo cambian los nombres del dominio.

---

### 20.1 Schema compartido (`packages/contracts`)

```typescript
// packages/contracts/src/jobs/create-job.schema.ts
import { z } from 'zod';

export const workModeSchema = z.enum(['remote', 'hybrid', 'onsite']);
export const jobStatusSchema = z.enum(['draft', 'open', 'closed', 'archived']);
export const creatableJobStatusSchema = z.enum(['draft', 'open']);

export const createJobRequestSchema = z.object({
  title: z.string().trim().min(3).max(120),
  department: z.string().trim().min(2).max(80),
  location: z.string().trim().min(2).max(120),
  workMode: workModeSchema,
  description: z.string().trim().min(20).max(10_000),
  status: creatableJobStatusSchema.default('draft'),
});

export type CreateJobRequest = z.infer<typeof createJobRequestSchema>;

export const jobResponseSchema = z.object({
  id: z.string(),
  tenantId: z.string(),
  title: z.string(),
  department: z.string(),
  location: z.string(),
  workMode: workModeSchema,
  description: z.string(),
  status: jobStatusSchema,
  createdBy: z.string(),
  createdAt: z.string(),
  updatedAt: z.string(),
  publishedAt: z.string().nullable(),
  closedAt: z.string().nullable(),
});

export type JobResponse = z.infer<typeof jobResponseSchema>;
```

---

### 20.2 Schema de Prisma

```prisma
// apps/api/prisma/schema.prisma (extracto relevante)
model Job {
  id                 String    @id @default(cuid())
  tenantId           String    @map("tenant_id")
  title              String
  department         String
  location           String
  workMode           String    @map("work_mode")
  description        String    @db.Text
  status             String    @default("draft")
  createdBy          String    @map("created_by")
  createdAt          DateTime  @default(now()) @map("created_at")
  updatedAt          DateTime  @updatedAt      @map("updated_at")
  publishedAt        DateTime? @map("published_at")
  closedAt           DateTime? @map("closed_at")

  tenant Tenant @relation(fields: [tenantId], references: [id], onDelete: Cascade)

  @@index([tenantId, status],    map: "idx_jobs_tenant_id_status")
  @@index([tenantId, createdAt], map: "idx_jobs_tenant_id_created_at")
  @@map("jobs")
}
```

---

### 20.3 Value Objects del Dominio

```typescript
// modules/jobs/domain/value-objects/job-id.vo.ts
import { ValueObject } from '@/shared/domain/value-object';
import { InvalidValueError } from '@/shared/domain/errors';

interface JobIdProps {
  value: string;
}

export class JobId extends ValueObject<JobIdProps> {
  private constructor(props: JobIdProps) {
    super(props);
  }

  public static create(value: string): JobId {
    if (typeof value !== 'string' || value.trim().length === 0) {
      throw new InvalidValueError('JobId cannot be empty');
    }
    return new JobId({ value });
  }

  public get value(): string {
    return this.props.value;
  }
}
```

```typescript
// modules/jobs/domain/value-objects/job-title.vo.ts
import { ValueObject } from '@/shared/domain/value-object';
import { JobTitleTooShortError, JobTitleTooLongError } from '../errors/job.errors';

const MIN_LENGTH = 3;
const MAX_LENGTH = 120;

interface JobTitleProps {
  value: string;
}

export class JobTitle extends ValueObject<JobTitleProps> {
  private constructor(props: JobTitleProps) {
    super(props);
  }

  public static create(raw: string): JobTitle {
    const value = raw.trim();
    if (value.length < MIN_LENGTH) {
      throw new JobTitleTooShortError(value.length, MIN_LENGTH);
    }
    if (value.length > MAX_LENGTH) {
      throw new JobTitleTooLongError(value.length, MAX_LENGTH);
    }
    return new JobTitle({ value });
  }

  public get value(): string {
    return this.props.value;
  }
}
```

```typescript
// modules/jobs/domain/value-objects/job-status.vo.ts
import { ValueObject } from '@/shared/domain/value-object';
import { InvalidJobStatusTransitionError } from '../errors/job.errors';

export const JOB_STATUSES = ['draft', 'open', 'closed', 'archived'] as const;
export type JobStatusValue = (typeof JOB_STATUSES)[number];

interface JobStatusProps {
  value: JobStatusValue;
}

export class JobStatus extends ValueObject<JobStatusProps> {
  private constructor(props: JobStatusProps) {
    super(props);
  }

  public static create(value: JobStatusValue): JobStatus {
    return new JobStatus({ value });
  }

  public static draft(): JobStatus {
    return new JobStatus({ value: 'draft' });
  }

  public static open(): JobStatus {
    return new JobStatus({ value: 'open' });
  }

  public get value(): JobStatusValue {
    return this.props.value;
  }

  public canTransitionTo(next: JobStatus): boolean {
    const transitions: Record<JobStatusValue, JobStatusValue[]> = {
      draft:    ['open', 'archived'],
      open:     ['closed', 'archived'],
      closed:   ['archived'],
      archived: [],
    };
    return transitions[this.value].includes(next.value);
  }

  public transitionTo(next: JobStatus): JobStatus {
    if (!this.canTransitionTo(next)) {
      throw new InvalidJobStatusTransitionError(this.value, next.value);
    }
    return next;
  }
}
```

```typescript
// modules/jobs/domain/value-objects/work-mode.vo.ts
import { ValueObject } from '@/shared/domain/value-object';

export const WORK_MODES = ['remote', 'hybrid', 'onsite'] as const;
export type WorkModeValue = (typeof WORK_MODES)[number];

interface WorkModeProps { value: WorkModeValue; }

export class WorkMode extends ValueObject<WorkModeProps> {
  private constructor(props: WorkModeProps) { super(props); }

  public static create(value: WorkModeValue): WorkMode {
    return new WorkMode({ value });
  }

  public get value(): WorkModeValue { return this.props.value; }
}
```

---

### 20.4 Errores de dominio

```typescript
// modules/jobs/domain/errors/job.errors.ts
import { DomainError } from '@/shared/domain/errors/domain-error';

export class JobTitleTooShortError extends DomainError {
  readonly code = 'JOB_TITLE_TOO_SHORT';
  constructor(public readonly actualLength: number, public readonly minLength: number) {
    super(`Job title must be at least ${minLength} characters. Got ${actualLength}.`, {
      actualLength,
      minLength,
    });
  }
}

export class JobTitleTooLongError extends DomainError {
  readonly code = 'JOB_TITLE_TOO_LONG';
  constructor(public readonly actualLength: number, public readonly maxLength: number) {
    super(`Job title must be at most ${maxLength} characters. Got ${actualLength}.`, {
      actualLength,
      maxLength,
    });
  }
}

export class InvalidJobStatusTransitionError extends DomainError {
  readonly code = 'INVALID_JOB_STATUS_TRANSITION';
  constructor(from: string, to: string) {
    super(`Cannot transition job status from "${from}" to "${to}".`, { from, to });
  }
}

export class JobNotFoundError extends DomainError {
  readonly code = 'JOB_NOT_FOUND';
  constructor(jobId: string) {
    super(`Job with id "${jobId}" not found.`, { jobId });
  }
}
```

---

### 20.5 Aggregate Root: `Job`

```typescript
// modules/jobs/domain/entities/job.entity.ts
import { AggregateRoot } from '@/shared/domain/aggregate-root';
import { Clock } from '@/shared/domain/ports/clock.port';
import { IdGenerator } from '@/shared/domain/ports/id-generator.port';
import { JobId } from '../value-objects/job-id.vo';
import { JobTitle } from '../value-objects/job-title.vo';
import { JobStatus } from '../value-objects/job-status.vo';
import { WorkMode } from '../value-objects/work-mode.vo';
import { JobCreatedEvent } from '../events/job-created.event';

interface JobProps {
  id: JobId;
  tenantId: string;
  title: JobTitle;
  department: string;
  location: string;
  workMode: WorkMode;
  description: string;
  status: JobStatus;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
  publishedAt: Date | null;
  closedAt: Date | null;
}

export interface CreateJobParams {
  tenantId: string;
  title: string;
  department: string;
  location: string;
  workMode: WorkMode;
  description: string;
  status: JobStatus;
  createdBy: string;
}

export class Job extends AggregateRoot<JobId> {
  private props: JobProps;

  private constructor(props: JobProps) {
    super(props.id);
    this.props = props;
  }

  public static create(
    params: CreateJobParams,
    clock: Clock,
    idGenerator: IdGenerator,
  ): Job {
    const now = clock.now();
    const id = JobId.create(idGenerator.generate());
    const title = JobTitle.create(params.title);

    const job = new Job({
      id,
      tenantId: params.tenantId,
      title,
      department: params.department,
      location: params.location,
      workMode: params.workMode,
      description: params.description,
      status: params.status,
      createdBy: params.createdBy,
      createdAt: now,
      updatedAt: now,
      publishedAt: params.status.value === 'open' ? now : null,
      closedAt: null,
    });

    job.addDomainEvent(
      new JobCreatedEvent({
        aggregateId: id.value,
        tenantId: params.tenantId,
        title: title.value,
        status: params.status.value,
        createdBy: params.createdBy,
        occurredAt: now,
        idGenerator,
      }),
    );

    return job;
  }

  public static reconstitute(props: JobProps): Job {
    return new Job(props);
  }

  public publish(clock: Clock): void {
    this.props.status = this.props.status.transitionTo(JobStatus.open());
    this.props.publishedAt = clock.now();
    this.props.updatedAt = clock.now();
  }

  public close(clock: Clock): void {
    this.props.status = this.props.status.transitionTo(
      JobStatus.create('closed'),
    );
    this.props.closedAt = clock.now();
    this.props.updatedAt = clock.now();
  }

  public toPrimitives(): {
    id: string;
    tenantId: string;
    title: string;
    department: string;
    location: string;
    workMode: string;
    description: string;
    status: string;
    createdBy: string;
    createdAt: Date;
    updatedAt: Date;
    publishedAt: Date | null;
    closedAt: Date | null;
  } {
    return {
      id: this.props.id.value,
      tenantId: this.props.tenantId,
      title: this.props.title.value,
      department: this.props.department,
      location: this.props.location,
      workMode: this.props.workMode.value,
      description: this.props.description,
      status: this.props.status.value,
      createdBy: this.props.createdBy,
      createdAt: this.props.createdAt,
      updatedAt: this.props.updatedAt,
      publishedAt: this.props.publishedAt,
      closedAt: this.props.closedAt,
    };
  }

  public get tenantId(): string { return this.props.tenantId; }
  public get id(): JobId { return this.props.id; }
  public get status(): JobStatus { return this.props.status; }
}
```

---

### 20.6 Domain Event

```typescript
// modules/jobs/domain/events/job-created.event.ts
import { DomainEvent } from '@/shared/domain/domain-event';
import { IdGenerator } from '@/shared/domain/ports/id-generator.port';

interface JobCreatedEventParams {
  aggregateId: string;
  tenantId: string;
  title: string;
  status: string;
  createdBy: string;
  occurredAt: Date;
  idGenerator: IdGenerator;
}

export class JobCreatedEvent extends DomainEvent {
  public readonly eventName = 'jobs.job.created';
  public readonly eventVersion = 1;
  public readonly title: string;
  public readonly status: string;
  public readonly createdBy: string;

  constructor(params: JobCreatedEventParams) {
    super({
      aggregateId: params.aggregateId,
      tenantId: params.tenantId,
      occurredAt: params.occurredAt,
      eventId: params.idGenerator.generate(),
    });
    this.title = params.title;
    this.status = params.status;
    this.createdBy = params.createdBy;
  }

  public toPayload(): Record<string, unknown> {
    return {
      jobId: this.aggregateId,
      tenantId: this.tenantId,
      title: this.title,
      status: this.status,
      createdBy: this.createdBy,
    };
  }
}
```

---

### 20.7 Puerto del Repositorio

```typescript
// modules/jobs/domain/ports/job.repository.ts
import { Job } from '../entities/job.entity';
import { JobId } from '../value-objects/job-id.vo';

export interface JobRepository {
  findById(tenantId: string, id: JobId): Promise<Job | null>;
  save(job: Job): Promise<void>;
}

export const JOB_REPOSITORY = Symbol('JobRepository');
```

---

### 20.8 Command y Handler (Application Layer)

```typescript
// modules/jobs/application/commands/create-job/create-job.command.ts
export class CreateJobCommand {
  constructor(
    public readonly title: string,
    public readonly department: string,
    public readonly location: string,
    public readonly workMode: 'remote' | 'hybrid' | 'onsite',
    public readonly description: string,
    public readonly status: 'draft' | 'open',
  ) {}
}
```

```typescript
// modules/jobs/application/commands/create-job/create-job.handler.ts
import { Job } from '../../../domain/entities/job.entity';
import { JobRepository } from '../../../domain/ports/job.repository';
import { WorkMode } from '../../../domain/value-objects/work-mode.vo';
import { JobStatus } from '../../../domain/value-objects/job-status.vo';
import { Clock } from '@/shared/domain/ports/clock.port';
import { IdGenerator } from '@/shared/domain/ports/id-generator.port';
import { EventBus } from '@/shared/domain/ports/event-bus.port';
import { UnitOfWork } from '@/shared/domain/ports/unit-of-work.port';
import { TenantContext } from '@/shared/domain/tenant-context';
import { CommandHandler } from '@/shared/application/command-handler';
import { JobDto } from '../../dtos/job.dto';
import { JobMapper } from '../../dtos/job.mapper';
import { CreateJobCommand } from './create-job.command';

export class CreateJobHandler implements CommandHandler<CreateJobCommand, JobDto> {
  constructor(
    private readonly jobRepository: JobRepository,
    private readonly unitOfWork: UnitOfWork,
    private readonly clock: Clock,
    private readonly idGenerator: IdGenerator,
    private readonly eventBus: EventBus,
  ) {}

  public async execute(
    context: TenantContext,
    command: CreateJobCommand,
  ): Promise<JobDto> {
    const job = Job.create(
      {
        tenantId: context.tenantId,
        title: command.title,
        department: command.department,
        location: command.location,
        workMode: WorkMode.create(command.workMode),
        description: command.description,
        status: JobStatus.create(command.status),
        createdBy: context.userId,
      },
      this.clock,
      this.idGenerator,
    );

    await this.unitOfWork.runInTransaction(async () => {
      await this.jobRepository.save(job);
      await this.eventBus.publishAll(job.pullDomainEvents());
    });

    return JobMapper.toDto(job);
  }
}
```

---

### 20.9 DTO y Mapper de aplicación

```typescript
// modules/jobs/application/dtos/job.dto.ts
export interface JobDto {
  id: string;
  tenantId: string;
  title: string;
  department: string;
  location: string;
  workMode: string;
  description: string;
  status: string;
  createdBy: string;
  createdAt: string;
  updatedAt: string;
  publishedAt: string | null;
  closedAt: string | null;
}
```

```typescript
// modules/jobs/application/dtos/job.mapper.ts
import { Job } from '../../domain/entities/job.entity';
import { JobDto } from './job.dto';

export class JobMapper {
  static toDto(job: Job): JobDto {
    const raw = job.toPrimitives();
    return {
      id: raw.id,
      tenantId: raw.tenantId,
      title: raw.title,
      department: raw.department,
      location: raw.location,
      workMode: raw.workMode,
      description: raw.description,
      status: raw.status,
      createdBy: raw.createdBy,
      createdAt: raw.createdAt.toISOString(),
      updatedAt: raw.updatedAt.toISOString(),
      publishedAt: raw.publishedAt?.toISOString() ?? null,
      closedAt: raw.closedAt?.toISOString() ?? null,
    };
  }
}
```

---

### 20.10 Repositorio concreto (Infrastructure)

```typescript
// modules/jobs/infrastructure/persistence/prisma-job.repository.ts
import { PrismaClient, Job as PrismaJob } from '@prisma/client';
import { JobRepository } from '../../domain/ports/job.repository';
import { Job } from '../../domain/entities/job.entity';
import { JobId } from '../../domain/value-objects/job-id.vo';
import { JobPersistenceMapper } from './job.mapper';

export class PrismaJobRepository implements JobRepository {
  constructor(private readonly prisma: PrismaClient) {}

  public async findById(tenantId: string, id: JobId): Promise<Job | null> {
    const raw = await this.prisma.job.findFirst({
      where: { id: id.value, tenantId },
    });
    return raw ? JobPersistenceMapper.toDomain(raw) : null;
  }

  public async save(job: Job): Promise<void> {
    const data = JobPersistenceMapper.toPersistence(job);
    await this.prisma.job.upsert({
      where: { id: data.id },
      create: data,
      update: {
        title: data.title,
        department: data.department,
        location: data.location,
        workMode: data.workMode,
        description: data.description,
        status: data.status,
        publishedAt: data.publishedAt,
        closedAt: data.closedAt,
      },
    });
  }
}
```

```typescript
// modules/jobs/infrastructure/persistence/job.mapper.ts
import type { Job as PrismaJob } from '@prisma/client';
import { Job } from '../../domain/entities/job.entity';
import { JobId } from '../../domain/value-objects/job-id.vo';
import { JobTitle } from '../../domain/value-objects/job-title.vo';
import { JobStatus, JobStatusValue } from '../../domain/value-objects/job-status.vo';
import { WorkMode, WorkModeValue } from '../../domain/value-objects/work-mode.vo';

export class JobPersistenceMapper {
  static toDomain(raw: PrismaJob): Job {
    return Job.reconstitute({
      id: JobId.create(raw.id),
      tenantId: raw.tenantId,
      title: JobTitle.create(raw.title),
      department: raw.department,
      location: raw.location,
      workMode: WorkMode.create(raw.workMode as WorkModeValue),
      description: raw.description,
      status: JobStatus.create(raw.status as JobStatusValue),
      createdBy: raw.createdBy,
      createdAt: raw.createdAt,
      updatedAt: raw.updatedAt,
      publishedAt: raw.publishedAt,
      closedAt: raw.closedAt,
    });
  }

  static toPersistence(job: Job): Omit<PrismaJob, 'tenant'> {
    const p = job.toPrimitives();
    return {
      id: p.id,
      tenantId: p.tenantId,
      title: p.title,
      department: p.department,
      location: p.location,
      workMode: p.workMode,
      description: p.description,
      status: p.status,
      createdBy: p.createdBy,
      createdAt: p.createdAt,
      updatedAt: p.updatedAt,
      publishedAt: p.publishedAt,
      closedAt: p.closedAt,
    };
  }
}
```

---

### 20.11 Controller HTTP

```typescript
// modules/jobs/interface/http/jobs.controller.ts
import type { Request, Response, NextFunction } from 'express';
import { CreateJobHandler } from '../../application/commands/create-job/create-job.handler';
import { CreateJobCommand } from '../../application/commands/create-job/create-job.command';
import { createJobRequestSchema } from '@ats/contracts/jobs';
import { ValidationError } from '@/shared/domain/errors/validation-error';

export class JobsController {
  constructor(private readonly createJobHandler: CreateJobHandler) {}

  public create = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const parsed = createJobRequestSchema.safeParse(req.body);
      if (!parsed.success) {
        throw ValidationError.fromZod(parsed.error);
      }
      const cmd = new CreateJobCommand(
        parsed.data.title,
        parsed.data.department,
        parsed.data.location,
        parsed.data.workMode,
        parsed.data.description,
        parsed.data.status,
      );

      const dto = await this.createJobHandler.execute(req.tenantContext, cmd);

      res
        .status(201)
        .location(`/api/v1/t/${req.tenantContext.tenantSlug}/jobs/${dto.id}`)
        .json({ data: dto, meta: { requestId: req.requestId } });
    } catch (err) {
      next(err);
    }
  };
}
```

```typescript
// modules/jobs/interface/http/jobs.routes.ts
import { Router } from 'express';
import { authMiddleware } from '@/shared/interface/middleware/auth';
import { tenantResolver } from '@/shared/interface/middleware/tenant-resolver';
import { rbac } from '@/shared/interface/middleware/rbac';
import { JobsController } from './jobs.controller';

export function buildJobsRouter(controller: JobsController): Router {
  const router = Router({ mergeParams: true });

  router.post(
    '/',
    authMiddleware(),
    tenantResolver(),
    rbac('jobs:create'),
    controller.create,
  );

  return router;
}
```

---

### 20.12 Unit tests del Handler

```typescript
// modules/jobs/application/commands/create-job/create-job.handler.spec.ts
import { CreateJobHandler } from './create-job.handler';
import { CreateJobCommand } from './create-job.command';
import { InMemoryJobRepository } from '@/test/fakes/in-memory-job.repository';
import { FakeClock } from '@/test/fakes/fake-clock';
import { FakeIdGenerator } from '@/test/fakes/fake-id-generator';
import { InMemoryEventBus } from '@/test/fakes/in-memory-event-bus';
import { NoopUnitOfWork } from '@/test/fakes/noop-unit-of-work';
import { JobTitleTooShortError } from '../../../domain/errors/job.errors';

describe('CreateJobHandler', () => {
  const tenantContext = {
    tenantId: 'tenant_1',
    tenantSlug: 'acme',
    userId: 'user_1',
    userRoles: ['recruiter'],
    requestId: 'req_test',
  };

  let repo: InMemoryJobRepository;
  let clock: FakeClock;
  let idGen: FakeIdGenerator;
  let bus: InMemoryEventBus;
  let uow: NoopUnitOfWork;
  let handler: CreateJobHandler;

  beforeEach(() => {
    repo = new InMemoryJobRepository();
    clock = new FakeClock(new Date('2026-01-15T10:00:00.000Z'));
    idGen = new FakeIdGenerator(['job_1', 'evt_1']);
    bus = new InMemoryEventBus();
    uow = new NoopUnitOfWork();
    handler = new CreateJobHandler(repo, uow, clock, idGen, bus);
  });

  describe('when the command is valid', () => {
    it('should create the job and publish JobCreatedEvent', async () => {
      // arrange
      const cmd = new CreateJobCommand(
        'Senior Backend Engineer',
        'Engineering',
        'Remote - LATAM',
        'remote',
        'We are looking for a senior backend engineer with 5+ years...',
        'draft',
      );

      // act
      const result = await handler.execute(tenantContext, cmd);

      // assert
      expect(result.id).toBe('job_1');
      expect(result.tenantId).toBe('tenant_1');
      expect(result.title).toBe('Senior Backend Engineer');
      expect(result.status).toBe('draft');

      const saved = await repo.findById('tenant_1', result.id);
      expect(saved).not.toBeNull();

      expect(bus.published).toHaveLength(1);
      expect(bus.published[0].eventName).toBe('jobs.job.created');
    });

    it('should set publishedAt when status is open', async () => {
      const cmd = new CreateJobCommand(
        'Senior Backend Engineer',
        'Engineering',
        'Remote',
        'remote',
        'Long enough description for validation to pass.',
        'open',
      );
      const result = await handler.execute(tenantContext, cmd);
      expect(result.publishedAt).toBe('2026-01-15T10:00:00.000Z');
    });
  });

  describe('when the title is too short', () => {
    it('should throw JobTitleTooShortError and not save', async () => {
      const cmd = new CreateJobCommand(
        'AB',
        'Engineering',
        'Remote',
        'remote',
        'Long enough description for validation.',
        'draft',
      );
      await expect(handler.execute(tenantContext, cmd)).rejects.toBeInstanceOf(
        JobTitleTooShortError,
      );
      expect(bus.published).toHaveLength(0);
    });
  });
});
```

---

### 20.13 Integration test

```typescript
// modules/jobs/__tests__/integration/create-job.integration.spec.ts
import request from 'supertest';
import { buildTestApp, TestHarness } from '@/test/setup/build-test-app';

describe('POST /api/v1/t/:tenantSlug/jobs (integration)', () => {
  let harness: TestHarness;

  beforeAll(async () => {
    harness = await buildTestApp();
  });

  afterAll(async () => {
    await harness.shutdown();
  });

  beforeEach(async () => {
    await harness.reset();
    await harness.seedTenant({ slug: 'acme', id: 'tenant_acme' });
    await harness.seedUser({
      id: 'user_1',
      tenantId: 'tenant_acme',
      roles: ['recruiter'],
    });
  });

  it('creates a job with status=draft and returns 201', async () => {
    const response = await request(harness.app)
      .post('/api/v1/t/acme/jobs')
      .set('Authorization', `Bearer ${harness.signUserToken('user_1', 'tenant_acme')}`)
      .set('X-Tenant-ID', 'tenant_acme')
      .send({
        title: 'Senior Backend Engineer',
        department: 'Engineering',
        location: 'Remote',
        workMode: 'remote',
        description: 'We are hiring a senior backend engineer with experience in Node.js',
        status: 'draft',
      });

    expect(response.status).toBe(201);
    expect(response.headers.location).toMatch(/\/api\/v1\/t\/acme\/jobs\/.+/);
    expect(response.body.data.title).toBe('Senior Backend Engineer');
    expect(response.body.data.status).toBe('draft');
  });

  it('rejects with 403 when X-Tenant-ID does not match path', async () => {
    const response = await request(harness.app)
      .post('/api/v1/t/acme/jobs')
      .set('Authorization', `Bearer ${harness.signUserToken('user_1', 'tenant_acme')}`)
      .set('X-Tenant-ID', 'tenant_OTHER')
      .send({
        title: 'Senior Backend Engineer',
        department: 'Engineering',
        location: 'Remote',
        workMode: 'remote',
        description: 'Description long enough for the validator.',
        status: 'draft',
      });

    expect(response.status).toBe(403);
    expect(response.body.error.code).toBe('TENANT_MISMATCH');
  });

  it('rejects with 400 when title is too short', async () => {
    const response = await request(harness.app)
      .post('/api/v1/t/acme/jobs')
      .set('Authorization', `Bearer ${harness.signUserToken('user_1', 'tenant_acme')}`)
      .set('X-Tenant-ID', 'tenant_acme')
      .send({
        title: 'AB',
        department: 'Engineering',
        location: 'Remote',
        workMode: 'remote',
        description: 'Description long enough for validation.',
        status: 'draft',
      });

    expect(response.status).toBe(400);
    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });

  it('rejects with 403 when user lacks jobs:create permission', async () => {
    await harness.seedUser({
      id: 'user_viewer',
      tenantId: 'tenant_acme',
      roles: ['viewer'],
    });
    const response = await request(harness.app)
      .post('/api/v1/t/acme/jobs')
      .set('Authorization', `Bearer ${harness.signUserToken('user_viewer', 'tenant_acme')}`)
      .set('X-Tenant-ID', 'tenant_acme')
      .send({
        title: 'Senior Backend Engineer',
        department: 'Engineering',
        location: 'Remote',
        workMode: 'remote',
        description: 'Description long enough for validation.',
        status: 'draft',
      });

    expect(response.status).toBe(403);
  });
});
```

---

### 20.14 Frontend: Server Action

```typescript
// apps/web/src/features/jobs/actions/create-job.action.ts
'use server';

import { auth } from '@/lib/auth';
import { apiClient } from '@/lib/api-client';
import { createJobRequestSchema } from '@ats/contracts/jobs';
import type { ActionResult } from '@/lib/action-result';
import type { JobResponse } from '@ats/contracts/jobs';
import { revalidatePath } from 'next/cache';

export async function createJobAction(
  tenantSlug: string,
  input: unknown,
): Promise<ActionResult<JobResponse>> {
  const session = await auth();
  if (!session?.user) {
    return { status: 'error', code: 'UNAUTHENTICATED', message: 'Not authenticated' };
  }

  const parsed = createJobRequestSchema.safeParse(input);
  if (!parsed.success) {
    return {
      status: 'error',
      code: 'VALIDATION_ERROR',
      message: 'Invalid input',
      fieldErrors: parsed.error.flatten().fieldErrors,
    };
  }

  const response = await apiClient.post<JobResponse>(
    `/t/${tenantSlug}/jobs`,
    parsed.data,
    { tenantSlug, session },
  );

  if (response.status === 'error') {
    return response;
  }

  revalidatePath(`/t/${tenantSlug}/jobs`);
  return { status: 'success', data: response.data };
}
```

---

### 20.15 Frontend: Componente del formulario

```typescript
// apps/web/src/features/jobs/components/create-job-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { useRouter } from 'next/navigation';
import { useTransition } from 'react';
import { createJobRequestSchema, type CreateJobRequest } from '@ats/contracts/jobs';
import { createJobAction } from '../actions/create-job.action';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Label } from '@/components/ui/label';
import { useToast } from '@/components/ui/use-toast';

interface CreateJobFormProps {
  tenantSlug: string;
}

export function CreateJobForm({ tenantSlug }: CreateJobFormProps): JSX.Element {
  const router = useRouter();
  const { toast } = useToast();
  const [isPending, startTransition] = useTransition();

  const form = useForm<CreateJobRequest>({
    resolver: zodResolver(createJobRequestSchema),
    defaultValues: {
      title: '',
      department: '',
      location: '',
      workMode: 'remote',
      description: '',
      status: 'draft',
    },
  });

  const onSubmit = form.handleSubmit((data) => {
    startTransition(async () => {
      const result = await createJobAction(tenantSlug, data);
      if (result.status === 'error') {
        toast({
          title: 'Error al crear la vacante',
          description: result.message,
          variant: 'destructive',
        });
        if (result.fieldErrors) {
          Object.entries(result.fieldErrors).forEach(([field, messages]) => {
            form.setError(field as keyof CreateJobRequest, {
              message: messages?.[0],
            });
          });
        }
        return;
      }
      toast({ title: 'Vacante creada', description: 'Lista para recibir postulaciones.' });
      router.push(`/t/${tenantSlug}/jobs/${result.data.id}`);
    });
  });

  return (
    <form onSubmit={onSubmit} className="space-y-6" data-testid="create-job-form">
      <div className="space-y-2">
        <Label htmlFor="title">Título</Label>
        <Input id="title" {...form.register('title')} data-testid="job-title-input" />
        {form.formState.errors.title && (
          <p className="text-sm text-destructive">{form.formState.errors.title.message}</p>
        )}
      </div>

      <div className="grid grid-cols-2 gap-4">
        <div className="space-y-2">
          <Label htmlFor="department">Área</Label>
          <Input id="department" {...form.register('department')} />
          {form.formState.errors.department && (
            <p className="text-sm text-destructive">{form.formState.errors.department.message}</p>
          )}
        </div>

        <div className="space-y-2">
          <Label htmlFor="location">Ubicación</Label>
          <Input id="location" {...form.register('location')} />
          {form.formState.errors.location && (
            <p className="text-sm text-destructive">{form.formState.errors.location.message}</p>
          )}
        </div>
      </div>

      <div className="space-y-2">
        <Label htmlFor="workMode">Modalidad</Label>
        <Select
          value={form.watch('workMode')}
          onValueChange={(v) => form.setValue('workMode', v as CreateJobRequest['workMode'])}
        >
          <SelectTrigger id="workMode" data-testid="job-workmode-select">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="remote">Remoto</SelectItem>
            <SelectItem value="hybrid">Híbrido</SelectItem>
            <SelectItem value="onsite">Presencial</SelectItem>
          </SelectContent>
        </Select>
      </div>

      <div className="space-y-2">
        <Label htmlFor="description">Descripción</Label>
        <Textarea id="description" rows={8} {...form.register('description')} />
        {form.formState.errors.description && (
          <p className="text-sm text-destructive">{form.formState.errors.description.message}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="status">Estado inicial</Label>
        <Select
          value={form.watch('status')}
          onValueChange={(v) => form.setValue('status', v as CreateJobRequest['status'])}
        >
          <SelectTrigger id="status">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="draft">Borrador</SelectItem>
            <SelectItem value="open">Abierta</SelectItem>
          </SelectContent>
        </Select>
      </div>

      <Button type="submit" disabled={isPending} data-testid="submit-create-job">
        {isPending ? 'Creando...' : 'Crear vacante'}
      </Button>
    </form>
  );
}
```

---

### 20.16 Página Next.js

```typescript
// apps/web/src/app/t/[tenantSlug]/jobs/new/page.tsx
import { auth } from '@/lib/auth';
import { notFound, redirect } from 'next/navigation';
import { hasPermissionInTenant } from '@/lib/permissions';
import { CreateJobForm } from '@/features/jobs/components/create-job-form';

interface PageProps {
  params: { tenantSlug: string };
}

export default async function NewJobPage({ params }: PageProps) {
  const session = await auth();
  if (!session?.user) {
    redirect(`/login?next=/t/${params.tenantSlug}/jobs/new`);
  }
  if (!hasPermissionInTenant(session, params.tenantSlug, 'jobs:create')) {
    notFound();
  }

  return (
    <main className="container mx-auto max-w-3xl py-8">
      <h1 className="text-2xl font-semibold mb-6">Nueva vacante</h1>
      <CreateJobForm tenantSlug={params.tenantSlug} />
    </main>
  );
}
```

---

### 20.17 E2E test (Playwright)

```typescript
// apps/web/e2e/specs/create-job.e2e.spec.ts
import { test, expect } from '@playwright/test';
import { loginAs } from '../fixtures/auth.fixture';

test.describe('Crear Vacante', () => {
  test('recruiter puede crear una vacante en estado borrador', async ({ page }) => {
    await loginAs(page, { email: 'recruiter@acme.test', tenantSlug: 'acme' });

    await page.goto('/t/acme/jobs/new');

    await page.getByTestId('job-title-input').fill('Senior Backend Engineer');
    await page.getByLabel('Área').fill('Engineering');
    await page.getByLabel('Ubicación').fill('Remote - LATAM');
    await page.getByTestId('job-workmode-select').click();
    await page.getByText('Remoto').click();
    await page.getByLabel('Descripción').fill(
      'Buscamos un ingeniero backend senior con experiencia en Node.js y PostgreSQL.',
    );

    await page.getByTestId('submit-create-job').click();

    await expect(page).toHaveURL(/\/t\/acme\/jobs\/[a-z0-9]+$/);
    await expect(page.getByText('Senior Backend Engineer')).toBeVisible();
    await expect(page.getByText('Borrador')).toBeVisible();
  });

  test('viewer no puede ver la opción de crear vacante', async ({ page }) => {
    await loginAs(page, { email: 'viewer@acme.test', tenantSlug: 'acme' });
    await page.goto('/t/acme/jobs/new');
    await expect(page).toHaveURL(/.*not-found|.*403/);
  });
});
```

---

### 20.18 Suscriptor al Domain Event (ejemplo: envío de email)

```typescript
// modules/notifications/interface/events/on-job-created.handler.ts
import type { JobCreatedEventPayload } from '@ats/contracts/events';
import { NotificationService } from '../../application/notification.service';
import { Logger } from '@/shared/domain/ports/logger.port';

export class OnJobCreatedHandler {
  public readonly eventName = 'jobs.job.created';

  constructor(
    private readonly notifications: NotificationService,
    private readonly logger: Logger,
  ) {}

  public async handle(payload: JobCreatedEventPayload): Promise<void> {
    this.logger.info(
      { tenantId: payload.tenantId, jobId: payload.jobId },
      'Handling job created event',
    );
    // No bloqueante: se envía vía cola de emails
    await this.notifications.enqueueJobCreatedNotification({
      tenantId: payload.tenantId,
      jobId: payload.jobId,
      title: payload.title,
      createdBy: payload.createdBy,
    });
  }
}
```

---

## 21. Referencia Rápida para el Agente

Cuando el agente reciba una instrucción, DEBE seguir este flujo:

```
1. ¿La instrucción está clara?
   └── NO → Preguntar al usuario.
   └── SÍ → continuar.

2. ¿Hay alguna regla MUST o MUST NOT que la instrucción viole?
   └── SÍ → Detenerse. Reportar el conflicto al usuario. NO proceder hasta confirmación.
   └── NO → continuar.

3. ¿Requiere agregar una dependencia no listada en sección 3?
   └── SÍ → Preguntar al usuario antes de agregarla.
   └── NO → continuar.

4. ¿Qué bounded context(s) toca?
   └── Identificar todos los módulos afectados.

5. ¿Hay un template relevante en la sección 20?
   └── SÍ → Usarlo como referencia estructural exacta.
   └── NO → Aplicar las reglas genéricas de la sección correspondiente.

6. Implementar siguiendo las reglas.

7. Ejecutar el checklist de la sección 18.

8. Reportar al usuario con:
   - Archivos creados/modificados.
   - Tests agregados.
   - Migraciones si aplica.
   - Pendientes / supuestos.
```

---

## Fin del documento

**Este documento es normativo.** Cualquier código del proyecto que contradiga estas reglas DEBE refactorizarse. Cualquier nueva funcionalidad DEBE agregarse siguiendo estas reglas.

Si el agente o un desarrollador considera que una regla debe cambiar, DEBE proponerse un cambio a este documento vía PR dedicado, con justificación técnica, antes de implementar el código que la contradice.

