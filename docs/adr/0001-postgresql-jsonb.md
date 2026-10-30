# ADR-001 — PostgreSQL 16 con JSONB como almacén primario

- **Estado**: aceptado
- **Fecha**: 2026-10-30
- **Contexto**: los dispositivos biomecánicos exportan estructuras heterogéneas
  (métricas planas, series temporales, metadatos). Se necesita integridad relacional
  para pacientes/evaluaciones y flexibilidad para `datos_raw`.

## Decisión

Usar PostgreSQL 16. El dominio relacional (pacientes, evaluaciones, adjuntos, usuarios)
vive en tablas normalizadas. Las métricas crudas y series cortas viven en columnas
`JSONB` con `schema_version`. Las series extensas se guardan como archivos C3D/Parquet en
object storage.

## Consecuencias

- Una sola base para dominio y datos semiestructurados; free tiers disponibles.
- Índices GIN cuando el volumen lo requiera.
- Riesgo de JSONB sin control: se mitiga con límites de tamaño y validación Pydantic.
- No se adopta TimescaleDB/InfluxDB en fase 0 para evitar complejidad operativa.

## Alternativas descartadas

- **MongoDB**: buena flexibilidad, pero menor integridad referencial y ecosistema SQL
  para investigación.
- **TimescaleDB**: excelente para series, pero agrega una extensión y costo operativo.
- **Archivos planos**: no ofrecen concurrencia, permisos ni trazabilidad.
