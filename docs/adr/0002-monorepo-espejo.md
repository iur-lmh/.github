# ADR-002 — Monorepo de orquestación con repos espejo

- **Estado**: aceptado
- **Fecha**: 2026-10-30
- **Contexto**: el equipo es pequeño y necesita desarrollar API, web y pipelines de forma
  coordinada, pero la organización exige repositorios independientes con CI propio y
  despliegues separados (Vercel, Render).

## Decisión

Mantener un workspace de orquestación con los tres módulos y la configuración `.github`.
Cada módulo contiene su propio `README.md`, `.env.example`, workflow y Dockerfile, y se
publica como repositorio independiente mediante `scripts/publish-repos.ps1`.

## Consecuencias

- Desarrollo local integrado con un solo `docker compose`.
- Publicación independiente sin perder historial (subtree split).
- Duplicación controlada de workflows raíz y de módulo.
- Requiere disciplina para no introducir dependencias cruzadas no declaradas.

## Alternativas descartadas

- **Tres repos separados desde el inicio**: fricción para cambios coordinados y para el
  entorno local.
- **Monorepo único desplegado como todo**: Vercel/Render esperan repos por servicio y
  complica permisos.
- **Submódulos Git**: DX frágil para un equipo pequeño.
