# MANIFEST — Ecosistema LabOS / IUR-LMH

Inventario oficial de repositorios, módulos, roles y servicios de la plataforma de
gestión biomecánica del **Laboratorio de Movimiento Humano (LMH)** del **Instituto
Universitario de Rehabilitación (IUR) — UCASAL**.

> Organización GitHub: [`iur-lmh`](https://github.com/iur-lmh)
> Estado: **Fase 0 — sustrato inicial**
> Última actualización: 2026-10-30

---

## 1. Repositorios

| Repositorio | Rol | Stack principal | Puerto local | Visibilidad |
|---|---|---|---|---|
| [`iur-lmh/.github`](https://github.com/iur-lmh/.github) | Configuración global, perfil institucional, plantillas de issues/PR, políticas y workflows compartidos | Markdown, YAML | — | Público |
| [`iur-lmh/labos-core`](https://github.com/iur-lmh/labos-core) | API REST, autenticación, RBAC, motor de ingestión y lógica clínica/biomecánica | Python 3.11+, FastAPI, SQLModel, PostgreSQL 16 | `8000` | Público |
| [`iur-lmh/labos-web`](https://github.com/iur-lmh/labos-web) | Portal de profesionales, mesa de I+D, portal de pacientes y hub público de investigación | Next.js (App Router), TypeScript, Tailwind, shadcn/ui, Recharts | `3000` | Público |
| [`iur-lmh/biomech-pipelines`](https://github.com/iur-lmh/biomech-pipelines) | Parsing de exportaciones heterogéneas, normalización, anonimización y exportación de datasets | Python 3.11+, pandas, Pydantic, Typer | CLI | Público |

### 1.1 Layout del workspace de desarrollo

Este workspace es el **monorepo de orquestación** que contiene los cuatro artefactos
versionables de forma independiente:

```text
lab-mh/
├── .github/                  # contenido del repo iur-lmh/.github
│   ├── ISSUE_TEMPLATE/
│   ├── workflows/
│   └── profile/README.md
├── labos-core/               # repo iur-lmh/labos-core
├── labos-web/                # repo iur-lmh/labos-web
├── biomech-pipelines/        # repo iur-lmh/biomech-pipelines
├── docs/                     # documentación transversal
├── scripts/                  # bootstrap y publicación de repos
├── docker-compose.yml        # stack local completo
├── MANIFEST.md
├── CONTEXT.md
└── STACK_INFO.md
```

Cada módulo conserva su propio `README.md`, `.env.example`, `Dockerfile` (cuando
aplica) y workflow de CI, de modo que puede escindirse a un repositorio independiente
sin cambios. El script `scripts/publish-repos.ps1` automatiza esa separación.

---

## 2. Módulos funcionales

### 2.1 `labos-core` — API y dominio

| Módulo | Ruta | Responsabilidad |
|---|---|---|
| API v1 | `app/api/v1/endpoints/` | Contratos REST: auth, pacientes, evaluaciones, investigaciones |
| Núcleo | `app/core/` | Configuración tipada, seguridad JWT, hashing, RBAC |
| Persistencia | `app/db/` | Engine, sesión, metadata, migraciones Alembic |
| Dominio | `app/models/` | `Paciente`, `Evaluacion`, `ArchivoAdjunto`, `Usuario` |
| Contratos | `app/schemas/` | DTOs Pydantic v2 de entrada/salida |
| Tests | `tests/` | Unitarios e integración con SQLite y PostgreSQL |

**Superficie REST inicial**

| Método | Ruta | Descripción | Roles |
|---|---|---|---|
| `POST` | `/api/v1/auth/login` | OAuth2 password flow → JWT | Público |
| `GET` | `/api/v1/auth/me` | Perfil autenticado | Todos |
| `POST` | `/api/v1/auth/usuarios` | Alta de usuario | Admin |
| `GET/POST` | `/api/v1/pacientes` | Listado / alta de pacientes | Profesional, Admin |
| `GET/PATCH/DELETE` | `/api/v1/pacientes/{id}` | Detalle / edición / baja | Profesional, Admin |
| `GET/POST` | `/api/v1/evaluaciones` | Listado / alta de evaluaciones | Profesional, Admin |
| `GET/DELETE` | `/api/v1/evaluaciones/{id}` | Detalle / baja | Profesional, Admin |
| `POST/GET` | `/api/v1/evaluaciones/{id}/archivos` | Registro / listado de adjuntos | Profesional, Admin |
| `GET` | `/api/v1/investigaciones/resumen` | Agregados para I+D | Investigador, Admin |
| `GET` | `/api/v1/investigaciones/dataset` | Dataset anonimizado (JSON/CSV) | Investigador, Admin |
| `GET` | `/health`, `/health/ready` | Liveness / readiness | Público |

### 2.2 `labos-web` — Experiencias de usuario

| Módulo | Ruta | Audiencia |
|---|---|---|
| Hub público de investigación | `app/(public)/` | Comunidad, pares académicos, alumnos |
| Login | `app/(auth)/login/` | Todos |
| Mesa de trabajo profesional | `app/(dashboard)/profesionales/` | Kinesiólogos, médicos, investigadores |
| Gestión de pacientes | `app/(dashboard)/profesionales/pacientes/` | Profesionales |
| Investigación / datasets | `app/(dashboard)/profesionales/investigacion/` | Investigadores |
| Portal de pacientes | `app/(dashboard)/pacientes/` | Pacientes / sujetos de estudio |

### 2.3 `biomech-pipelines` — Ingeniería de datos

| Módulo | Ruta | Responsabilidad |
|---|---|---|
| Adaptadores | `src/biomech_pipelines/parsers/` | Biobit, mDurance, C3D, CSV genérico, media (MP4) |
| Normalización | `src/biomech_pipelines/normalize.py` | Modelo canónico de sesión biomecánica |
| Privacidad | `src/biomech_pipelines/anonymize.py` | Seudonimización HMAC, limpieza de PII, desplazamiento temporal |
| Exportación | `src/biomech_pipelines/export.py` | CSV, JSONL, Parquet (opcional) |
| CLI | `src/biomech_pipelines/cli.py` | `biomech parse / anonymize / export / inspect` |

---

## 3. Roles y permisos (RBAC)

| Rol | Código | Puede |
|---|---|---|
| Administrador | `admin` | Gestionar usuarios, pacientes, evaluaciones, configuración y datasets |
| Profesional / Investigador | `profesional` | Gestionar pacientes y evaluaciones; consultar y exportar datasets anonimizados |
| Paciente / Sujeto | `paciente` | Ver únicamente sus propias evaluaciones y documentos |

Reglas transversales:

- El acceso a datos clínicos identificables requiere rol `admin` o `profesional`.
- Las exportaciones de investigación **siempre** se sirven seudonimizadas.
- Un paciente solo puede consultar registros cuyo `paciente_id` coincida con su vínculo.
- Los endpoints de escritura exigen JWT válido; las lecturas también, salvo el hub público.

---

## 4. Servicios e infraestructura

| Servicio | Proveedor inicial | Free tier | Uso |
|---|---|---|---|
| Base de datos | Supabase / Neon (PostgreSQL 16) | Sí | Datos clínicos y JSONB |
| Object storage | Cloudflare R2 | 10 GB | C3D, MP4, PDF, CSV |
| Object storage local | S3Mock (MinIO opcional) | Local | Desarrollo y respaldo |
| Frontend | Vercel | Hobby | `labos-web` |
| Backend | Render / Railway / VPS | Limitado | `labos-core` |
| CI/CD | GitHub Actions | 2.000 min/mes | Lint, tipos, tests, build |
| Auth | JWT propio (HS256) + RBAC | — | Sesiones stateless |
| Observabilidad | Sentry free + logs estructurados | Sí | Errores y trazas |

### 4.1 Variables de entorno por servicio

| Variable | Servicio | Obligatoria | Descripción |
|---|---|---|---|
| `DATABASE_URL` | core | Sí | DSN PostgreSQL (`postgresql+psycopg://...`) |
| `SECRET_KEY` | core | Sí | Firma JWT (≥ 32 bytes aleatorios) |
| `CORS_ORIGINS` | core | Sí | Orígenes permitidos, separados por coma |
| `ADMIN_EMAIL` / `ADMIN_PASSWORD` | core | No | Semilla de administrador inicial |
| `S3_ENDPOINT_URL` / `S3_ACCESS_KEY_ID` / `S3_SECRET_ACCESS_KEY` / `S3_BUCKET` | core | No | Cloudflare R2 / S3 compatible |
| `NEXT_PUBLIC_API_URL` | web | Sí | URL pública de la API |
| `LABOS_API_INTERNAL_URL` | web | No | URL interna para SSR/proxy |
| `NEXT_PUBLIC_APP_NAME` | web | No | Branding |
| `BIOMECH_ANON_SALT` | pipelines | Sí en export | Sal HMAC para seudonimizar |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` | compose | Sí | Credenciales locales |

> La lista canónica y comentada vive en cada `.env.example`.

---

## 5. Convenciones

- **Ramas**: `main` protegida; trabajo en `feat/*`, `fix/*`, `docs/*`, `chore/*`.
- **Commits**: [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`…).
- **Python**: Ruff (lint + format), mypy, pytest. Línea 100.
- **TypeScript**: ESLint, Prettier, `tsc --noEmit`, Vitest.
- **API**: versionado por prefijo `/api/v1`; cambios incompatibles crean `/api/v2`.
- **Datos**: toda exportación pasa por `biomech-pipelines`; nunca se versionan datos reales.
- **Secretos**: solo variables de entorno; `.env` está prohibido en git.

---

## 6. Propiedad y soporte

| Área | Responsable |
|---|---|
| Dirección institucional | Laboratorio de Movimiento Humano — IUR / UCASAL |
| Mantenimiento técnico | Equipo de desarrollo IUR-LMH |
| Contacto | laboratoriodemovimientoucasal@ucasal.edu.ar |
| Seguridad | Ver [`SECURITY.md`](.github/SECURITY.md) |

Los repositorios se publican bajo licencia **MIT** (ver `LICENSE`). Los datos de
pacientes **no** forman parte de este repositorio ni de ningún repositorio público.
