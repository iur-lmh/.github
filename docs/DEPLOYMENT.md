# Despliegue con costo inicial $0

Arquitectura de referencia: Vercel (web) + Render (API) + Supabase/Neon (PostgreSQL) +
Cloudflare R2 (archivos) + GitHub Actions (CI).

> Los límites de los free tiers cambian. Verificar precios y cuotas antes de producción.

## 1. Base de datos — Supabase o Neon

1. Crear proyecto PostgreSQL 16 en la región más cercana (São Paulo / us-east).
2. Copiar la connection string **pooled** para la API:
   `postgresql+psycopg://USER:PASSWORD@HOST:6543/postgres?sslmode=require`
3. Para migraciones usar la conexión directa (puerto 5432) si el pooler no soporta DDL.
4. Configurar backups automáticos y alertas de uso.

```bash
cd labos-core
DATABASE_URL="postgresql+psycopg://..." alembic upgrade head
```

## 2. Object storage — Cloudflare R2

1. Crear bucket privado `labos-archivos` (o `labos-prod`).
2. Crear token S3 con permiso de lectura/escritura solo sobre ese bucket.
3. Configurar CORS para el dominio de Vercel:
   - Allowed origins: `https://labos.vercel.app`, `http://localhost:3000`
   - Allowed methods: `GET, PUT, HEAD`
   - Allowed headers: `content-type, x-amz-*`
4. Variables en la API:

```env
S3_ENDPOINT_URL=https://<accountid>.r2.cloudflarestorage.com
S3_ACCESS_KEY_ID=...
S3_SECRET_ACCESS_KEY=...
S3_BUCKET=labos-archivos
S3_REGION=auto
```

> Para desarrollo local, usar el perfil `storage` de Docker Compose (S3Mock). MinIO
> también es compatible: apuntar `S3_ENDPOINT_URL` al endpoint propio.

## 3. API — Render (Docker)

1. New → Web Service → conectar `iur-lmh/labos-core`.
2. Runtime: Docker. Health check: `/health`. Región: Oregon o São Paulo.
3. Variables de entorno:

```env
ENVIRONMENT=production
DATABASE_URL=postgresql+psycopg://...
SECRET_KEY=<48 bytes aleatorios>
CORS_ORIGINS=https://<dominio-web>
ADMIN_EMAIL=<email institucional>
ADMIN_PASSWORD=<contraseña fuerte, rotar tras el primer login>
S3_ENDPOINT_URL=...
S3_ACCESS_KEY_ID=...
S3_SECRET_ACCESS_KEY=...
S3_BUCKET=labos-archivos
S3_REGION=auto
```

4. El contenedor ejecuta `alembic upgrade head` y luego Uvicorn.
5. En free tier el servicio duerme tras inactividad: usar un ping externo cada 14 min o
   aceptar el arranque en frío.

### Alternativa VPS

```bash
git clone https://github.com/iur-lmh/labos-core.git
cd labos-core
cp .env.example .env   # editar
docker build -t labos-core .
docker run -d --env-file .env -p 8000:8000 --name labos-core labos-core
```

Detrás de Caddy/Nginx con TLS (Let's Encrypt). Configurar `restart: unless-stopped`.

## 4. Web — Vercel

1. Importar `iur-lmh/labos-web`.
2. Framework preset: Next.js. Build: `npm run build`.
3. Variables:

```env
NEXT_PUBLIC_API_URL=https://labos-api.onrender.com
LABOS_API_INTERNAL_URL=https://labos-api.onrender.com
NEXT_PUBLIC_APP_NAME=LabOS LMH
```

4. Dominio: `labos.iur-lmh.org` o subdominio institucional (requiere DNS).
5. Verificar que las cookies funcionen: API y web deben compartir sitio o usar el proxy
   de Route Handlers (incluido).

## 5. CI/CD

Cada repo incluye `.github/workflows/ci.yml`. Configurar en GitHub:

- Settings → Branches → protection rule para `main`:
  - Require pull request + 1 approval.
  - Require status checks: `lint`, `typecheck`, `test`, `build`.
  - Require branch up to date.
- Settings → Secrets → `CODECOV_TOKEN` (opcional).
- Dependabot habilitado (`.github/dependabot.yml`).

## 6. Checklist de producción

- [ ] `SECRET_KEY` única, ≥ 32 bytes, rotada.
- [ ] Contraseña de admin inicial cambiada.
- [ ] `ENVIRONMENT=production` (desactiva `/docs`? evaluar).
- [ ] CORS limitado al dominio real.
- [ ] HTTPS forzado; cookies `Secure`.
- [ ] Backups automáticos + prueba de restauración.
- [ ] Bucket privado; CORS restringido.
- [ ] Logs sin PII; Sentry configurado.
- [ ] Consentimientos informados digitalizados.
- [ ] Comité de ética aprobó el flujo de investigación.
- [ ] Plan de respuesta a incidentes y contacto de seguridad.
- [ ] Límites de free tier monitoreados.

## 7. Respaldos

```bash
# Dump comprimido y cifrado
pg_dump "$DATABASE_URL" | gzip | age -r <clave-pública> > labos-$(date +%F).sql.gz.age

# Subir a R2
aws s3 cp labos-$(date +%F).sql.gz.age s3://labos-backups/ \
  --endpoint-url "$S3_ENDPOINT_URL"
```

Retención sugerida: 7 diarios, 4 semanales, 12 mensuales. Restaurar en un entorno staging
al menos una vez por trimestre.
