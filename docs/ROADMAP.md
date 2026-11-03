# Hoja de ruta

Cada hito tiene un entregable y un criterio de cierre verificable. Los hitos se ordenan por dependencia, no por fecha.

## Decisiones de arquitectura (ADR resumidos)

**ADR-001. Descarga desde Airflow, no desde Databricks.**
La lista de dominios permitidos de Free Edition no es pública y, según la comunidad, varía por región y ha cambiado sin aviso. Airflow descarga cada fichero en local y lo sube a un Volume de Unity Catalog mediante la Files API (hasta 5.000 MiB por petición). Si H0 confirma acceso saliente desde serverless, la descarga directa se documenta como alternativa comparada.

**ADR-002. Orquestación con `DatabricksRunNowOperator`.**
Los jobs se definen en Databricks con Asset Bundles desplegadas desde local; Airflow solo los dispara. Se evita `DatabricksSubmitRunOperator`, cuya documentación no muestra configuración serverless. Alternativa descartada: Lakeflow Jobs como único orquestador (suficiente técnicamente, pero Airflow es requisito habitual en las ofertas objetivo).

**ADR-003. Esquema unificado explícito.**
Los ficheros de la TLC cambian tipos, nombres y columnas entre años (ver [proyecto.md](proyecto.md)). Bronce conserva el dato tal como llega más metadatos de ingesta; plata aplica un esquema unificado con conversión de tipos y nombres en minúscula.

**ADR-004. Núcleo y prueba de escala separados.**
Núcleo: taxis amarillos y verdes desde 2019. HVFHV se reserva para H6, donde se mide el techo de Free Edition.

## Decisiones pendientes del autor

- [ ] Alcance temporal (propuesta: desde 2019, coincidiendo con el inicio de HVFHV).
- [ ] Segunda fuente: Open-Meteo histórico o NOAA (Central Park).
- [ ] Ejecución de dbt: desde Airflow en local (propuesta) o como tarea dbt en un job de Databricks.

---

## H0. Verificaciones técnicas

- [ ] Región del workspace.
- [ ] Descarga de un Parquet de la TLC desde un notebook serverless; anotar resultado o error exacto.
- [ ] Repetir la prueba tras verificar con LinkedIn, si está disponible.
- [ ] Subida de un fichero a un Volume con la Files API desde la máquina local.
- [ ] Conexión de dbt-databricks al SQL warehouse 2X-Small con token de acceso personal.
- [ ] Llamada a la API de jobs desde local.
- [ ] Despliegue de una Asset Bundle mínima desde local.
- [ ] Disponibilidad de `system.billing.usage`.
- [ ] Acceso a la fuente meteorológica elegida.

**Entregable:** `docs/fase0.md` con una tabla prueba / resultado / evidencia.
**Cierre:** ADR-001 y ADR-002 confirmados o corregidos con evidencia.

## H1. Esqueleto del repositorio

- [ ] Estructura: `ingestion/`, `dbt/`, `airflow/`, `bundles/`, `docs/`.
- [ ] Airflow 3 con Docker Compose y un DAG vacío.
- [ ] GitHub Actions: `ruff`, `sqlfluff`, `dbt parse`.
- [ ] Gestión de secretos fuera del repositorio (`.env` ignorado, conexiones de Airflow).

**Cierre:** la integración continua pasa y Airflow arranca en local.

## H2. Ingesta y capa bronce

- [ ] Registro de ficheros: fuente, mes, tamaño, suma de comprobación, estado, fecha de carga.
- [ ] Descarga y subida al Volume idempotentes (no se repite un fichero ya registrado con la misma suma).
- [ ] Carga a bronce con columnas de fichero de origen y fecha de ingesta.
- [ ] Carga histórica por lotes respetando la cuota diaria.

**Cierre:** cargar dos veces el mismo mes produce el mismo número de filas; histórico 2019–actual cargado.

## H3. Capa plata y calidad

- [ ] Esquema unificado (ADR-003).
- [ ] Reglas de limpieza: distancias e importes negativos, fechas fuera del mes del fichero, zonas inexistentes.
- [ ] Modelos incrementales (`merge`) en dbt.
- [ ] Pruebas genéricas y propias; frescura de la fuente.
- [ ] Tabla de zonas como seed.

**Cierre:** `dbt build` en verde y porcentaje de filas descartadas medido y documentado.

## H4. Capa oro y modelo analítico

- [ ] Modelo dimensional: hechos de viaje; dimensiones de zona, fecha, hora y tipo de pago.
- [ ] Ingesta de la fuente meteorológica con el mismo patrón que H2.
- [ ] Agregados para las preguntas de negocio.
- [ ] `dbt docs` generado.

**Cierre:** preguntas de negocio respondidas con SQL sobre oro.

## H5. Orquestación completa y observabilidad

- [ ] DAG mensual con `catchup`, reintentos y dependencias ingesta → bronce → dbt.
- [ ] Avisos de fallo por webhook.
- [ ] Tabla de registro de ejecuciones: duración, filas, resultado de pruebas.

**Cierre:** ejecución de extremo a extremo sin intervención manual y un fallo provocado que genera aviso.

## H6. Prueba de escala con HVFHV

- [ ] Ingesta progresiva por lotes.
- [ ] Medición de tiempos, consumo y punto de agotamiento de la cuota.

**Cierre:** tabla de mediciones y techo de Free Edition documentado.

## H7. Análisis, documentación y publicación

- [ ] Conclusiones de negocio.
- [ ] README bilingüe (EN/ES) con diagrama de arquitectura.
- [ ] ADR completos y sección de limitaciones.

**Cierre:** el proyecto se reproduce siguiendo únicamente el README.
