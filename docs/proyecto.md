# Proyecto 1 del portfolio: pipeline de datos cloud

**Estado:** definición revisada. Los puntos marcados como *pendiente de verificar* se resuelven en el hito H0 (ver [ROADMAP.md](ROADMAP.md)).
**Fecha:** 3 de noviembre de 2026

---

## 1. Objetivo

### Objetivo del proyecto
Construir un pipeline de datos de extremo a extremo, sobre datos públicos reales y de gran volumen, que demuestre el dominio del stack Python, SQL, dbt, Airflow y Databricks, y que produzca información útil para la toma de decisiones.

### Objetivo profesional
Servir como pieza principal del portfolio para acceder a puestos de ingeniero de datos cloud (con un componente de analista de datos para cubrir también ese tipo de ofertas), junto con la certificación de Databricks y las candidaturas en curso.

### Criterio de éxito
El proyecto no se valora por su originalidad, sino por la calidad de su ejecución:

- Un pipeline completo que se ejecuta de forma orquestada y repetible.
- Pruebas de calidad de datos y registro de ejecuciones.
- Mediciones reales (volumen, tiempos, consumo de recursos) y no afirmaciones sin respaldo.
- Una documentación que explique las decisiones de diseño, sus alternativas y las limitaciones encontradas.

### Principio de trabajo
El proyecto lo construye el autor; Claude actúa como guía para comprender el funcionamiento del stack, sin entregar componentes ya hechos.

---

## 2. Alcance

### Dataset principal
**NYC TLC Trip Record Data** (registros de viajes de taxis y vehículos de alquiler con conductor de Nueva York).

Datos verificados el 3 de noviembre de 2026 directamente sobre los ficheros publicados (`d37ci6vzurychx.cloudfront.net`):

| Fichero | Tamaño | Filas |
|---|---|---|
| Amarillos 2019-01 | — | 7.696.617 |
| Amarillos 2023-01 | — | 3.066.766 |
| Amarillos 2025-06 | 73,5 MB | 4.322.960 |
| Amarillos 2026-09 | 67,4 MB | 3.876.044 |
| Verdes 2025-06 | 1,2 MB | — |
| HVFHV 2024-01 | 472,8 MB | — |
| HVFHV 2025-06 | 490,8 MB | — |

- Formato Parquet, publicación mensual. El último mes disponible a la fecha de revisión es 2026-09.
- El conjunto de amarillos y verdes desde 2019 se sitúa en el orden de cientos de millones de filas. El volumen alto corresponde a HVFHV (unos 0,5 GB al mes, decenas de GB desde 2019).
- El bucket `s3://nyc-tlc` del registro de datos abiertos de AWS sigue listado, pero exige cuenta de AWS para acceso por S3; no aporta ventaja frente a la descarga directa.

### Variaciones de esquema detectadas (yellow)

| Columna | 2019-01 | 2023-01 | 2025-06 / 2026-09 |
|---|---|---|---|
| `VendorID`, `PULocationID`, `DOLocationID` | int64 | int64 | int32 |
| `passenger_count`, `RatecodeID` | double | double | int64 |
| `store_and_fwd_flag` | string | string | large_string |
| `airport_fee` | `airport_fee` (null) | `airport_fee` (double) | `Airport_fee` (double) |
| `cbd_congestion_fee` | — | — | double (desde 2025) |
| `request_source` | — | — | large_string (2026-09; no documentado en la página de la TLC) |

Consecuencia: la ingesta no puede apoyarse en la fusión automática de esquemas. Se requiere un esquema unificado explícito, con conversión de tipos y normalización de nombres.

### Segunda fuente (enriquecimiento)
Datos meteorológicos. Candidatas: API histórica de Open-Meteo o NOAA (estación de Central Park). *Pendiente de decidir en H0.*

### Dentro del alcance

1. **Ingesta** de ficheros mensuales con carga incremental e idempotente, controlada por un registro de ficheros.
2. **Almacenamiento por capas** (bronce, plata y oro) en Databricks, sobre Unity Catalog.
3. **Transformación y modelado** con dbt, con pruebas de calidad de datos.
4. **Orquestación** con Airflow.
5. **Calidad y observabilidad:** validaciones, avisos ante fallos y registro de ejecuciones.
6. **Capa analítica** con preguntas de negocio concretas:
   - Demanda por zona y franja horaria.
   - Evolución de tarifas y propinas.
   - Relación con la meteorología.
7. **Prueba de escala** con HVFHV para medir el techo real de Free Edition.
8. **Documentación bilingüe (EN/ES)** con decisiones de diseño, mediciones y limitaciones.

### Fuera del alcance (por ahora)

- Streaming en tiempo real.
- Modelos de machine learning (ya cubiertos por el proyecto de previsión de demanda).
- Servicios de AWS u otros clouds, que se incorporarán solo si el plan lo requiere.
- El proyecto Stock-Agent-Sys, que queda como segundo proyecto, y Nexo, que queda como línea futura.

### Restricciones de Databricks Free Edition

Según la documentación oficial:

- Solo cómputo serverless; no se admiten configuraciones de cómputo personalizadas.
- Acceso saliente a internet restringido a un conjunto de dominios de confianza no publicado. La verificación con LinkedIn, si está disponible para la cuenta, habilita acceso saliente.
- Un único SQL warehouse de tamaño 2X-Small, un máximo de 5 tareas de job concurrentes y un pipeline activo por tipo en Lakeflow.
- Si se supera la cuota, el cómputo se apaga durante el resto del día (los datos se conservan).
- No puede usarse con fines comerciales, lo que es compatible con un portfolio personal.

Observaciones de la comunidad (no oficiales): la lista de dominios permitidos varía según la región y ha cambiado sin aviso; el despliegue de Asset Bundles desde el propio workspace falla porque no puede descargar Terraform, mientras que desde una máquina local funciona.

Consecuencias de diseño:
- La descarga de los ficheros se realiza fuera de Databricks (desde Airflow) y se suben a un Volume mediante la Files API. Ver ADR en [ROADMAP.md](ROADMAP.md).
- La carga histórica se hace por lotes para no agotar la cuota diaria.
- Free Edition no factura; las mediciones se expresan en tiempos y consumo de recursos (y en DBU si `system.billing.usage` está disponible).

---

## 3. Stack

| Capa | Tecnología | Función en el proyecto |
|---|---|---|
| Lenguajes | Python, SQL | Ingesta, utilidades y consultas |
| Plataforma de datos | Databricks Free Edition (Unity Catalog, Volumes, Delta) | Almacenamiento y cómputo por capas |
| Transformación | dbt Core + dbt-databricks (SQL warehouse) | Modelado, pruebas y documentación |
| Orquestación | Airflow 3 (Docker Compose local) | Descarga, subida al Volume y disparo de jobs |
| Despliegue Databricks | Asset Bundles (desde local) | Definición versionada de jobs |
| Integración continua | GitHub Actions | Lint de Python y SQL, `dbt parse` |
| Control de versiones | Git / GitHub | Repositorio público del portfolio |

Integración Airflow–Databricks: se usa `DatabricksRunNowOperator` sobre jobs definidos con Asset Bundles. No se usa `DatabricksSubmitRunOperator`, cuya documentación no muestra configuración serverless. *Pendiente de verificar en H0:* llamada a la API de jobs con token de acceso personal.

*Posible ampliación:* AWS u otra plataforma, si las ofertas objetivo lo justifican.

---

## 4. Hitos

Ver [ROADMAP.md](ROADMAP.md).

---

## Fuentes

- Databricks Free Edition limitations: https://docs.databricks.com/aws/en/getting-started/free-edition-limitations
- Free Edition (verificación con LinkedIn): https://docs.databricks.com/gcp/en/getting-started/free-edition-limitations
- Comunidad, lista de dominios: https://community.databricks.com/t5/databricks-free-edition-help/whitelist-for-outbound-network-access/td-p/146591
- Comunidad, bloqueo repentino: https://community.databricks.com/t5/data-engineering/free-edition-outbound-internet-suddenly-blocked-error/td-p/121832
- Comunidad, Asset Bundles: https://community.databricks.com/t5/administration-architecture/asset-bundle-on-free-edition/td-p/127236
- Jobs serverless: https://docs.databricks.com/aws/jobs/run-serverless-jobs
- Files API: https://docs.databricks.com/aws/en/files/files-api
- DatabricksRunNowOperator: https://airflow.apache.org/docs/apache-airflow-providers-databricks/stable/operators/run_now.html
- DatabricksSubmitRunOperator: https://airflow.apache.org/docs/apache-airflow-providers-databricks/stable/operators/submit_run.html
- dbt-databricks: https://docs.getdbt.com/docs/core/connect-data-platform/databricks-setup
- NYC TLC Trip Record Data: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- Registro de datos abiertos de AWS: https://registry.opendata.aws/nyc-tlc-trip-records-pds
