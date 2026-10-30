# Roadmap

## Fase 0 — Sustrato (actual)

- [x] Documentación de contexto, manifiesto y stack.
- [x] API FastAPI con modelos, esquemas, JWT/RBAC y CRUD base.
- [x] Frontend Next.js con hub público y dashboards iniciales.
- [x] Pipelines de parsing y anonimización con CLI.
- [x] Docker Compose (PostgreSQL + API + storage S3/web opcionales).
- [x] CI de lint, tipos y tests.
- [x] Plantillas de issues/PR y perfil de organización.
- [ ] Repos publicados en GitHub.
- [ ] Validación con datos reales de una muestra piloto.

## Fase 1 — Operación clínica mínima (4–6 semanas)

- [ ] Uploads reales a R2 con URLs prefirmadas.
- [ ] Visor de curvas fuerza-tiempo y series EMG.
- [ ] Informe PDF por evaluación.
- [ ] Auditoría de accesos (`audit_log`).
- [ ] Rate limiting y bloqueo por intentos fallidos.
- [ ] Migraciones automáticas en CI con PostgreSQL.
- [ ] Seed de catálogo de protocolos.

## Fase 2 — Ingestión robusta (6–8 semanas)

- [ ] Cola asíncrona para parsing y normalización.
- [ ] Adaptadores validados de Biobit y mDurance.
- [ ] Detección automática de formato y versión.
- [ ] Catálogo de dispositivos y calibraciones.
- [ ] Deduplicación por hash SHA-256.
- [ ] Reintentos y estados de ingesta.

## Fase 3 — Investigación (8 semanas)

- [ ] Constructor de datasets con filtros y diccionario.
- [ ] Exportación Parquet/CSV con manifiesto de reproducibilidad.
- [ ] Estadísticas descriptivas y control de k-anonimato.
- [ ] Versionado de datasets y DOI (Zenodo/OSF).
- [ ] Integración con notebooks R/Python.
- [ ] Hub público conectado a publicaciones reales.

## Fase 4 — Escala y gobernanza

- [ ] SSO institucional (opcional).
- [ ] Multi-sede / multi-laboratorio.
- [ ] Firma digital de informes.
- [ ] Panel de administración y métricas de uso.
- [ ] Revisión de seguridad externa.
- [ ] Política de retención y eliminación de datos.

## Backlog priorizado

| Prioridad | Ítem | Impacto |
|---|---|---|
| P0 | Upload a R2 + validación de archivos | Desbloquea operación real |
| P0 | Auditoría de accesos | Cumplimiento |
| P0 | Backup/restore probado | Continuidad |
| P1 | Visor de curvas | Valor clínico |
| P1 | Informe PDF | Entrega al paciente |
| P1 | Adaptadores reales Biobit/mDurance | Ingestión |
| P2 | Constructor de datasets | Investigación |
| P2 | Portal paciente completo | Experiencia |
| P3 | SSO y multi-sede | Escala |
