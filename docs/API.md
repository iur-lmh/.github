# API REST — LabOS v1

Base URL local: `http://localhost:8000/api/v1`
OpenAPI interactivo: `http://localhost:8000/docs`

## Autenticación

```http
POST /api/v1/auth/login
Content-Type: application/x-www-form-urlencoded

username=admin@labos.ar&password=...
```

```json
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "expires_in": 28800
}
```

Usar `Authorization: Bearer <token>` en el resto de los endpoints.

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/auth/login` | público | Emite JWT |
| GET | `/auth/me` | autenticado | Perfil actual |
| POST | `/auth/usuarios` | admin | Crea usuario |
| GET | `/auth/usuarios` | admin | Lista usuarios |

## Pacientes

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/pacientes?q=&skip=0&limit=50` | admin, profesional | Búsqueda paginada |
| POST | `/pacientes` | admin, profesional | Alta |
| GET | `/pacientes/{id}` | admin, profesional | Detalle |
| PATCH | `/pacientes/{id}` | admin, profesional | Edición parcial |
| DELETE | `/pacientes/{id}` | admin | Baja con cascada |

```json
POST /pacientes
{
  "nombre": "Ana",
  "apellido": "Gómez",
  "fecha_nacimiento": "1994-03-12",
  "dni_anonimizado": "a3f1c9e2b7d4",
  "metadata": {"deporte": "running", "categoria": "amateur"}
}
```

## Evaluaciones

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/evaluaciones?paciente_id=&tipo_protocolo=&dispositivo=&skip=&limit=` | admin, profesional | Listado filtrado |
| POST | `/evaluaciones` | admin, profesional | Alta |
| GET | `/evaluaciones/{id}` | admin, profesional | Detalle |
| DELETE | `/evaluaciones/{id}` | admin | Baja |
| GET | `/evaluaciones/{id}/archivos` | admin, profesional | Adjuntos |
| POST | `/evaluaciones/{id}/archivos` | admin, profesional | Registra adjunto |

```json
POST /evaluaciones
{
  "paciente_id": "0f7c...",
  "tipo_protocolo": "salto_vertical",
  "fecha": "2026-10-29T14:30:00-03:00",
  "dispositivo_origen": "Biobit",
  "datos_raw": {
    "schema_version": "1.0",
    "metricas": {"altura_salto_cm": 34.2, "tiempo_vuelo_ms": 512}
  }
}
```

## Investigaciones

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/investigaciones/resumen` | admin, profesional | Agregados por protocolo/dispositivo |
| GET | `/investigaciones/dataset?formato=json&tipo_protocolo=&dispositivo=&desde=&hasta=` | admin, profesional | Dataset seudonimizado |

`formato=csv` devuelve `text/csv` con `Content-Disposition`.
El dataset excluye nombre, apellido y fecha de nacimiento; conserva
`dni_anonimizado` como seudónimo y `datos_raw`.

## Salud

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/health` | Liveness (no toca DB) |
| GET | `/health/ready` | Readiness (consulta `SELECT 1`) |

## Errores

Formato estándar:

```json
{"detail": "Paciente no encontrado"}
```

| Código | Significado |
|---|---|
| 400 | Datos inválidos o duplicados |
| 401 | Token ausente/expirado/credenciales |
| 403 | Rol insuficiente |
| 404 | Recurso inexistente |
| 409 | Conflicto de unicidad |
| 422 | Error de validación Pydantic |
| 500 | Error interno (sin detalles sensibles) |

## Paginación

Respuesta:

```json
{
  "items": [],
  "total": 0,
  "skip": 0,
  "limit": 50
}
```

Límite máximo: `limit <= 200`.
