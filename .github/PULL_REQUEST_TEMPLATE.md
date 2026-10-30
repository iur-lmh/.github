# Resumen

<!-- Explicá en 2-3 líneas qué cambia y por qué. -->

## Tipo de cambio

- [ ] Nueva funcionalidad
- [ ] Corrección de error
- [ ] Refactor / deuda técnica
- [ ] Documentación
- [ ] CI / infraestructura
- [ ] Ingestión de nuevo dispositivo

## Cambios

<!-- Lista de cambios relevantes, endpoints, migraciones o pantallas afectadas. -->

## Cómo probar

<!-- Pasos concretos. Si aplica, comandos y datos sintéticos usados. -->

```bash
# ejemplo
cd labos-core && pytest
```

## Checklist

- [ ] Los tests nuevos/actualizados pasan localmente.
- [ ] `lint`, `format` y `typecheck` pasan.
- [ ] Actualicé `.env.example` si agregué variables.
- [ ] Agregué migración de Alembic si cambié el esquema.
- [ ] Actualicé la documentación (README, `docs/`, OpenAPI) si cambió el contrato.
- [ ] No incluí datos de pacientes, secretos, tokens ni archivos `.env`.
- [ ] Las exportaciones o logs nuevos no exponen PII.
- [ ] El cambio respeta los roles RBAC definidos.
- [ ] Si toqué un adaptador de dispositivo, agregué una muestra sintética y un test.

## Privacidad y seguridad

<!-- ¿El cambio accede a datos clínicos? ¿Cómo se protege? ¿Requiere revisión de seguridad? -->

## Capturas / evidencia

<!-- Para cambios de UI, incluir antes/después. Usar datos de demostración. -->
