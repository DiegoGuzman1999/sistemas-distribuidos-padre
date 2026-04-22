# ADR-004 — Tres Ambientes Independientes con Docker Compose

**Fecha:** 2026-04-07
**Estado:** Aceptado
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El proyecto requería soportar un flujo de trabajo con ramas Git (`dev`, `qa`, `main`). Era necesario decidir cómo desplegar cada rama de forma que los ambientes no se interfirieran entre sí y pudieran correr simultáneamente en la misma máquina.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Un solo docker-compose con variables de entorno | Cambiar variables para apuntar a distinto ambiente |
| Kubernetes con namespaces | Un namespace por ambiente |
| Tres archivos docker-compose independientes | Un archivo por ambiente con puertos y redes separados |

## Decisión

Se eligieron **tres archivos Docker Compose independientes**, uno por ambiente:

| Archivo | Ambiente | Puertos |
|---|---|---|
| `docker-compose.dev.yml` | DEV | 3000, 5000, 5001, 5432 |
| `docker-compose.qa.yml` | QA | 3001, 5010, 5011, 5433 |
| `docker-compose.main.yml` | MAIN | 3002, 5020, 5021, 5434 |

Cada archivo define su propia red Docker (`red-dev`, `red-qa`, `red-main`) y su propia base de datos (`inventario_db_dev`, `inventario_db_qa`, `inventario_db_main`).

## Justificación

- Es la solución más directa y sin dependencias externas (no requiere Kubernetes ni herramientas adicionales).
- Permite levantar los 3 ambientes simultáneamente en la misma máquina sin conflictos de puertos ni datos.
- La separación por redes Docker garantiza que los contenedores de un ambiente no puedan comunicarse con los de otro.
- Se alinea con el flujo Git `dev → qa → main`: cada rama tiene su ambiente dedicado.

## Consecuencias

- **Positivas:** Aislamiento total entre ambientes, 12 contenedores corriendo simultáneamente sin interferencia, fácil de entender y operar.
- **Negativas:** Tres archivos de configuración a mantener en paralelo. Un cambio en la estructura del servicio debe replicarse en los tres archivos.
