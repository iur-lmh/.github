# Política de seguridad

El Laboratorio de Movimiento Humano (IUR — UCASAL) trata datos clínicos sensibles. La
seguridad y la privacidad son requisitos de diseño, no características opcionales.

## Versiones soportadas

El proyecto está en fase 0 (pre-1.0). Solo la rama `main` recibe correcciones de
seguridad.

| Versión | Soporte |
|---|---|
| `main` | Sí |
| Tags anteriores | No (hasta estabilizar 1.0) |

## Cómo reportar una vulnerabilidad

**No abras un issue público.** Usá uno de estos canales:

1. **GitHub Security Advisories** (preferido):
   <https://github.com/iur-lmh/.github/security/advisories/new>
2. Email: `laboratoriodemovimientoucasal@ucasal.edu.ar` con asunto
   `[SEGURIDAD] LabOS`.

Incluí:

- Descripción del problema y componente afectado.
- Pasos de reproducción con datos **sintéticos**.
- Impacto estimado (confidencialidad, integridad, disponibilidad).
- Versión/commit y entorno.
- Si lo tenés, una sugerencia de mitigación.

**Nunca incluyas datos reales de pacientes, credenciales, tokens ni historias clínicas
en el reporte.** Si el hallazgo involucra un dato real, describilo de forma abstracta y
coordinemos un canal seguro.

## Compromisos

| Etapa | Plazo objetivo |
|---|---|
| Acuse de recibo | 72 h hábiles |
| Evaluación inicial | 7 días |
| Corrección crítica | 30 días |
| Divulgación coordinada | Acordada con quien reporta |

## Alcance

- API `labos-core`, autenticación JWT, RBAC y exposición de datos.
- Portal `labos-web`, manejo de cookies, proxy y XSS/CSRF.
- Pipelines de anonimización y riesgo de reidentificación.
- Infraestructura y CI (secretos, dependencias, permisos).
- Documentación que induzca a prácticas inseguras.

## Fuera de alcance

- Ingeniería social.
- Denegación de servicio sobre free tiers.
- Vulnerabilidades en servicios de terceros (Vercel, Supabase, R2) sin impacto directo
  en este código; reportarlas a su proveedor.
- Falta de controles organizacionales que no dependan del software.

## Prácticas del proyecto

- Secretos solo en variables de entorno; `.env` ignorado.
- Contraseñas con Argon2id; JWT con expiración y firma HS256.
- Cookies httpOnly y `SameSite=Lax`; sin tokens en `localStorage`.
- Exportaciones seudonimizadas con sal custodiada.
- Dependencias actualizadas vía Dependabot.
- Revisión de PR obligatoria en `main`.
- Logs sin PII.

Gracias por ayudar a proteger a las personas cuyos datos sostienen la investigación.
