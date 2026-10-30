# Contribuir a LabOS

Gracias por sumarte. Este documento aplica a todos los repositorios de la organización
[`iur-lmh`](https://github.com/iur-lmh).

## Principios irrenunciables

1. **Privacidad**: nunca commitear datos de pacientes, PII, historias clínicas ni
   muestras reales. Usar datos sintéticos.
2. **Trazabilidad**: toda evaluación conserva dispositivo, fecha y hash de archivo.
3. **Reproducibilidad**: los cambios de pipeline registran versión y tests.
4. **Costo**: no introducir servicios que rompan el objetivo de free tier sin justificar.
5. **Respeto**: seguir el [Código de Conducta](CODE_OF_CONDUCT.md).

## Entorno de desarrollo

Requisitos: Docker 24+, Python 3.11+, Node 20+.

```bash
git clone https://github.com/iur-lmh/<repo>.git
cd <repo>
```

### Monorepo local (si trabajás con los cuatro módulos)

```bash
cp .env.example .env
docker compose up --build
```

### Backend

```bash
cd labos-core
python -m venv .venv
. .venv/Scripts/activate   # Windows
pip install -r requirements-dev.txt
cp .env.example .env
alembic upgrade head
uvicorn app.main:app --reload
```

### Frontend

```bash
cd labos-web
npm install
cp .env.example .env.local
npm run dev
```

### Pipelines

```bash
cd biomech-pipelines
python -m venv .venv
. .venv/Scripts/activate
pip install -e ".[dev]"
biomech --help
```

## Flujo de trabajo

1. Abrí un issue (bug, feature o dispositivo) usando las plantillas.
2. Creá una rama desde `main`:
   - `feat/descripcion-corta`
   - `fix/descripcion-corta`
   - `docs/...`, `chore/...`, `ci/...`
3. Hacé commits pequeños con [Conventional Commits](https://www.conventionalcommits.org/):
   `feat(api): agregar filtro por dispositivo`.
4. Abrí un Pull Request y completá la plantilla.
5. Asegurate de que todos los checks pasen antes de pedir revisión.

## Calidad mínima (Definition of Done)

| Requisito | Backend | Frontend | Pipelines |
|---|---|---|---|
| Lint | `ruff check .` | `npm run lint` | `ruff check .` |
| Formato | `ruff format --check .` | `npm run format:check` | `ruff format --check .` |
| Tipos | `mypy app` | `npm run typecheck` | `mypy src` |
| Tests | `pytest` | `npm run test` | `pytest` |
| Build | Docker | `npm run build` | `pip install -e .` |

Además:

- Migraciones Alembic para cambios de esquema.
- `.env.example` actualizado para variables nuevas.
- Documentación/OpenAPI actualizados si cambia el contrato.
- Tests con datos sintéticos; nunca archivos reales.
- Sin secretos en el diff.

## Nuevos dispositivos biomecánicos

1. Abrí un issue con la plantilla **Ingestión de un nuevo dispositivo**.
2. Agregá un adaptador en `biomech-pipelines/src/biomech_pipelines/parsers/`.
3. Documentá columnas, unidades y supuestos en el docstring y el README.
4. Agregá una muestra sintética en `tests/fixtures/` y tests de contrato.
5. Validá que la salida cumpla `SesionNormalizada` y no contenga PII.
6. Actualizá la tabla de formatos soportados.

## Revisión de PR

- Un approval mínimo; dos para cambios de seguridad o esquema.
- No se mergea con checks rojos.
- Los cambios destructivos de API requieren nueva versión (`/api/v2`).
- Los cambios de privacidad requieren revisión explícita del responsable del laboratorio.

## Reportes de seguridad

Ver [SECURITY.md](SECURITY.md). No abras issues públicos para vulnerabilidades.

## Licencia

Al contribuir aceptás que tu aporte se publique bajo la licencia MIT del repositorio.
