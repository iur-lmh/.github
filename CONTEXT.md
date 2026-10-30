# CONTEXT — Plataforma LabOS / IUR-LMH

Documento de contexto operativo para personas y agentes que trabajen en el ecosistema.
Responde **por qué** existe la plataforma, **para quién**, **bajo qué restricciones** y
**qué queda fuera de alcance**.

---

## 1. Contexto institucional

El **Laboratorio de Movimiento Humano (LMH)** pertenece al **Instituto Universitario de
Rehabilitación (IUR)** de la **Universidad Católica de Salta (UCASAL)**, Argentina.

El laboratorio produce datos biomecánicos de distinta naturaleza:

- Plataformas de salto y fuerza (Biobit).
- Unidades inerciales y análisis de movimiento (mDurance, IMUs).
- Captura óptica y datos cinemáticos (C3D).
- Electromiografía (EMG) y señales fisiológicas.
- Video de sesiones (MP4), informes (PDF) y planillas (CSV/XLSX).

Hoy esos datos viven fragmentados entre dispositivos, computadoras personales, carpetas
compartidas y planillas manuales. Esto limita la trazabilidad clínica, la reproducibilidad
de la investigación y la continuidad del equipo.

## 2. Propósito

Construir una **API de ingestión** y un **Data Lake biomecánico unificado** que:

1. Reemplace el almacenamiento manual y el software fragmentado.
2. Preserve la trazabilidad entre paciente → evaluación → archivo → métrica.
3. Permita a profesionales consultar resultados en una interfaz ágil.
4. Permita a investigadores exportar datasets anonimizados y reproducibles.
5. Habilite un hub público de publicaciones, pre-prints y líneas de investigación.

## 3. Autonomía e independencia tecnológica

El laboratorio desarrolla su infraestructura **con independencia de las áreas IT
centralizadas** de la universidad, sin depender de servidores internos ni de compras.

Restricciones autoimpuestas:

- **Costo inicial $0/mes**: Vercel Free, Supabase/Neon Free, Cloudflare R2 Free,
  GitHub Actions Free, Render/Railway Free o VPS existente.
- **Open Source First**: componentes y librerías con licencias permisivas.
- **Portabilidad**: PostgreSQL estándar y object storage S3-compatible; sin lock-in.
- **Operación mínima**: un desarrollador debe poder desplegar con `docker compose` y
  variables de entorno.
- **Degradación elegante**: si el storage no está configurado, la API sigue operativa y
  registra metadatos; los archivos se habilitan cuando existan credenciales.

## 4. Usuarios y necesidades

| Usuario | Necesita | Dolor actual |
|---|---|---|
| Kinesiólogo / médico | Cargar una evaluación y ver resultados en minutos | Múltiples exportaciones y planillas |
| Investigador | Descargar series crudas y métricas listas para R/Python | Datos dispersos, sin diccionario |
| Paciente / sujeto | Ver su evolución y sus informes | Sin acceso directo |
| Dirección / laboratorio | Saber qué se midió, cuándo y con qué dispositivo | Sin inventario central |
| Comunidad académica | Conocer publicaciones y líneas de trabajo | Información no publicada |

## 5. Principios de diseño

1. **Privacidad primero.** Los datos clínicos son sensibles. Seudonimización en origen,
   mínimo privilegio y separación entre identificadores y métricas.
2. **Interoperabilidad científica.** Salidas tidy/long, CSV/Parquet/JSON, compatibles con
   R y Python. Diccionario de datos versionado.
3. **Contratos explícitos.** Pydantic v2 y OpenAPI como fuente de verdad; cambios
   incompatibles requieren nueva versión de API.
4. **Modularidad.** API, web y pipelines evolucionan por separado y se comunican por
   contratos estables (REST + archivos).
5. **Trazabilidad.** Toda evaluación conserva dispositivo de origen, fecha, archivo y
   hash SHA-256; toda exportación conserva la versión del pipeline.
6. **Accesibilidad y agilidad.** Interfaz en español, navegación clara, tiempos de carga
   bajos, uso en notebooks y móviles.
7. **Costo y sostenibilidad.** Sin servicios que generen facturación sorpresa; límites de
   free tier documentados.

## 6. Marco legal y ético (Argentina)

- **Ley 25.326** de Protección de los Datos Personales: los datos de salud son
  sensibles; se requiere consentimiento informado y medidas de seguridad proportionates.
- **Ley 26.529** de Derechos del Paciente: confidencialidad de la historia clínica.
- **Consentimiento informado** para uso clínico y para investigación; el consentimiento
  de investigación debe permitir retiro y especificar finalidad.
- **Anonimización vs. seudonimización**: un hash estable sigue siendo dato personal si
  permite reidentificación con la sal. La sal se custodia fuera del dataset.
- **Transferencia internacional**: verificar destino de Supabase/R2/Vercel y cláusulas
  contractuales antes de cargar datos reales.
- **Datos de menores**: requieren consentimiento de tutores y controles adicionales.

> Este documento no constituye asesoramiento legal. Antes de producción con datos reales,
> el IUR debe validar el flujo con su área legal y el comité de ética.

## 7. Alcance de la fase 0 (este repositorio)

Incluido:

- Documentación de contexto, manifiesto y stack.
- API FastAPI ejecutable con modelos, esquemas, auth JWT/RBAC y endpoints base.
- Frontend Next.js con portal público, login y dashboards iniciales.
- Pipelines de parsing/anonimización con CLI y tests.
- Docker Compose para PostgreSQL + API (+ storage S3 y web opcionales).
- CI de lint, tipos y tests; plantillas de issues/PR.

Fuera de alcance (backlog):

- Integraciones propietarias certificadas con Biobit/mDurance.
- Firma digital de informes y receta electrónica.
- Historia clínica completa (solo evaluaciones biomecánicas).
- Facturación, turnos y gestión administrativa.
- Aplicación móvil nativa.
- Machine learning en producción (solo exploración offline).
- Cumplimiento formal certificado (ISO 27001, HIPAA).

## 8. Métricas de éxito

| Métrica | Objetivo 12 meses |
|---|---|
| Evaluaciones cargadas con archivo asociado | ≥ 80 % |
| Tiempo de carga de una sesión | < 5 min |
| Exportaciones reproducibles con versión de pipeline | 100 % |
| Incidentes de privacidad | 0 |
| Costo mensual de infraestructura | $0 (free tiers) |
| Cobertura de tests backend | ≥ 80 % en rutas críticas |

## 9. Glosario

| Término | Definición |
|---|---|
| **Evaluación** | Sesión de medición de un paciente con un protocolo y dispositivo |
| **Protocolo** | Procedimiento estandarizado: salto, marcha, EMG, fuerza, etc. |
| **Datos raw** | Métricas y series tal como las exporta el dispositivo, sin interpretación |
| **Dataset** | Conjunto anonimizado de evaluaciones preparado para análisis |
| **Seudónimo** | Identificador estable derivado por HMAC de un identificador real |
| **R2 / S3** | Object storage compatible con S3 para archivos binarios (R2, MinIO, s3mock) |
| **Mesa de I+D** | Espacio de trabajo para explorar, filtrar y exportar datasets |

## 10. Decisiones de arquitectura iniciales

| ID | Decisión | Motivo |
|---|---|---|
| ADR-001 | PostgreSQL 16 + JSONB | Relacional para integridad, JSONB para series heterogéneas |
| ADR-002 | FastAPI + SQLModel | OpenAPI automático, tipado, velocidad de desarrollo |
| ADR-003 | JWT stateless con RBAC | Simple, compatible con Vercel/Render, sin sesiones en DB |
| ADR-004 | Object storage S3-compatible | Evita binarios en DB; portable entre R2 y MinIO |
| ADR-005 | Pipelines como CLI independiente | Reutilizable en notebooks, CI y tareas batch |
| ADR-006 | Exportación seudonimizada obligatoria | Reduce riesgo legal y facilita colaboración |
| ADR-007 | Monorepo de orquestación + repos espejo | Desarrollo integrado, publicación independiente |

Las decisiones futuras se registran en `docs/adr/`.
