# Fase 0: verificaciones técnicas

Registro de pruebas sobre la cuenta real de Databricks Free Edition. Cada fila se rellena con el resultado **exacto** (mensaje de error completo si falla), la fecha y la evidencia. Una prueba sin evidencia no cuenta como verificada.

**Fecha de inicio:** _
**Región del workspace:** _

| # | Prueba | Qué decide | Resultado | Evidencia | Fecha |
|---|---|---|---|---|---|
| 1 | Región del workspace y tipo de cuenta | Lista de dominios aplicable | | | |
| 2 | Descarga de un Parquet de la TLC desde un notebook serverless (antes de verificar con LinkedIn) | ADR-001 | | | |
| 3 | Misma prueba tras verificar con LinkedIn (si está disponible) | ADR-001 | | | |
| 4 | Subida de un Parquet a un Volume con la Files API desde local | ADR-001 | | | |
| 5 | Llamada a la API de jobs desde local con token de acceso personal | ADR-002 | | | |
| 6 | Despliegue de una Asset Bundle mínima desde local | ADR-002 | | | |
| 7 | Ejecución de ese job desde local con `run-now` (simula Airflow) | ADR-002 | | | |
| 8 | dbt-databricks: `dbt debug` contra el SQL warehouse 2X-Small | Ejecución de dbt | | | |
| 9 | Disponibilidad de `system.billing.usage` | Medición de consumo | | | |
| 10 | Acceso a la fuente meteorológica elegida (Open-Meteo / NOAA) | Segunda fuente | | | |

## Conclusiones

- ADR-001 (descarga desde Airflow): _ confirmado / corregido, porque _
- ADR-002 (`DatabricksRunNowOperator`): _ confirmado / corregido, porque _
- Ejecución de dbt: _
- Segunda fuente: _
- Problemas inesperados: _
