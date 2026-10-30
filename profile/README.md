# IUR · Laboratorio de Movimiento Humano

**Plataforma abierta de gestión biomecánica e investigación clínica.**

Organización de desarrollo del **Laboratorio de Movimiento Humano (LMH)** del
**Instituto Universitario de Rehabilitación (IUR) — Universidad Católica de Salta
(UCASAL)**, Argentina.

Construimos infraestructura para reemplazar planillas dispersas y software fragmentado
por una API de ingestión y un *data lake* biomecánico unificado: trazable, interoperable
con R/Python y con la privacidad clínica como primer requisito.

[![License: MIT](https://img.shields.io/badge/license-MIT-15576e.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.11%2B-15576e.svg)](https://www.python.org/)
[![Next.js](https://img.shields.io/badge/Next.js-App_Router-15576e.svg)](https://nextjs.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-15576e.svg)](https://www.postgresql.org/)
[![Costo](https://img.shields.io/badge/infra-free_tier-0d9488.svg)](#infraestructura)

## Repositorios

| Repositorio | Descripción | Stack |
|---|---|---|
| [**labos-core**](https://github.com/iur-lmh/labos-core) | API REST, autenticación JWT/RBAC, ingestión y lógica clínica | FastAPI · SQLModel · PostgreSQL 16 |
| [**labos-web**](https://github.com/iur-lmh/labos-web) | Portal de profesionales, mesa de I+D, portal de pacientes y hub público | Next.js · TypeScript · Tailwind · Recharts |
| [**biomech-pipelines**](https://github.com/iur-lmh/biomech-pipelines) | Parsing de Biobit/mDurance/C3D/CSV, anonimización y datasets | Python · pandas · Pydantic · Typer |
| [**`.github`**](https://github.com/iur-lmh/.github) | Contexto, manifiesto, plantillas y políticas compartidas | Markdown · YAML |

## Arquitectura

```text
Dispositivos              Plataforma                         Investigación
━━━━━━━━━━━━              ━━━━━━━━━━                         ━━━━━━━━━━━━━
Biobit      ─┐       ┌─→ API FastAPI ─→ PostgreSQL 16      ┌→ Datasets CSV/Parquet
mDurance    ─┼─→ CLI ─┤     (JWT/RBAC)      + JSONB        │   seudonimizados
C3D / MP4   ─┘       └─→ Object storage ─→ R2 / S3 ─────┘   (R / Python)
CSV / PDF                 (C3D, MP4, PDF)
                              ↑
                        Next.js (Vercel)
```

## Principios

- **Privacidad por diseño**: datos clínicos sensibles, mínimo privilegio, seudonimización
  obligatoria en exportaciones, sin PII en logs.
- **Open source first**: licencias permisivas, sin lock-in, portable entre proveedores.
- **Interoperabilidad**: JSONB + archivos C3D/Parquet/CSV, salidas tidy para R y Python.
- **Costo inicial $0**: Vercel, Supabase/Neon, Cloudflare R2 y GitHub Actions free tier.
- **Trazabilidad**: paciente → evaluación → archivo (SHA-256) → métrica → dataset.

## Empezar

```bash
git clone https://github.com/iur-lmh/labos-core.git
# o el módulo que necesites; cada repo es independiente

# Stack local completo (monorepo de desarrollo)
docker compose up --build
```

- API: `http://localhost:8000` · Swagger: `/docs`
- Web: `http://localhost:3000`
- Pipelines: `biomech --help`

## Documentación

- [Manifiesto e inventario](https://github.com/iur-lmh/.github/blob/main/MANIFEST.md)
- [Contexto institucional](https://github.com/iur-lmh/.github/blob/main/CONTEXT.md)
- [Stack tecnológico](https://github.com/iur-lmh/.github/blob/main/STACK_INFO.md)
- [Arquitectura](https://github.com/iur-lmh/.github/blob/main/docs/ARCHITECTURE.md)
- [Modelo de datos](https://github.com/iur-lmh/.github/blob/main/docs/DATA_MODEL.md)
- [Despliegue](https://github.com/iur-lmh/.github/blob/main/docs/DEPLOYMENT.md)

## Seguridad y datos

Este espacio **no contiene datos de pacientes**. Los ejemplos y tests usan datos
sintéticos. Para reportar una vulnerabilidad, seguí
[SECURITY.md](https://github.com/iur-lmh/.github/blob/main/SECURITY.md) y no abras un
issue público.

## Contacto

Laboratorio de Movimiento Humano — IUR / UCASAL
Salta, Argentina
[laboratoriodemovimientoucasal@ucasal.edu.ar](mailto:laboratoriodemovimientoucasal@ucasal.edu.ar)
[www.ucasal.edu.ar/laboratorio-de-movimiento-humano](https://www.ucasal.edu.ar/laboratorio-de-movimiento-humano)

> Documentación y código en desarrollo activo (fase 0). Los límites de los free tiers y
> los protocolos clínicos deben validarse antes de un despliegue con datos reales.
