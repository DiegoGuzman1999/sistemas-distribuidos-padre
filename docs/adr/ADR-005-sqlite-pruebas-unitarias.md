# ADR-005 — Usar SQLite en Memoria para Pruebas Unitarias

**Fecha:** 2026-04-07
**Estado:** Aceptado
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

Al implementar las pruebas unitarias con pytest para los backends Flask, era necesario decidir contra qué base de datos correr las pruebas. El sistema en producción usa PostgreSQL dentro de Docker, pero requerir Docker y PostgreSQL para ejecutar pruebas unitarias aumenta la fricción del proceso de desarrollo.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| PostgreSQL real (via Docker) | Levantar el stack completo antes de cada ejecución de pruebas |
| PostgreSQL en CI/CD | Usar un servicio PostgreSQL en el pipeline de integración continua |
| SQLite en memoria | Base de datos liviana integrada en Python, sin instalación ni Docker |

## Decisión

Se eligió **SQLite en memoria** (`sqlite:///:memory:`) con `StaticPool` para garantizar que todas las operaciones compartan la misma conexión durante cada prueba.

```python
application.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
application.config['SQLALCHEMY_ENGINE_OPTIONS'] = {
    'connect_args': {'check_same_thread': False},
    'poolclass': StaticPool,
}
```

Se modificó `create_app()` para aceptar un parámetro `test_config` opcional, permitiendo inyectar la configuración de pruebas sin alterar el comportamiento en producción.

## Justificación

- Las pruebas corren sin necesidad de Docker, PostgreSQL ni ninguna dependencia externa.
- Cada prueba parte de una base de datos limpia (`db.create_all()` / `db.drop_all()`), garantizando aislamiento total.
- SQLAlchemy es compatible con SQLite y PostgreSQL, por lo que los modelos y queries funcionan igual en ambos motores para los casos cubiertos.
- Reduce el tiempo de ejecución de las pruebas de minutos a segundos.

## Consecuencias

- **Positivas:** Pruebas rápidas, independientes de infraestructura, ejecutables en cualquier máquina con Python.
- **Negativas:** SQLite no soporta todas las funciones de PostgreSQL (ej. tipos de dato específicos, algunas funciones de agregación). Si se agregan queries avanzadas en el futuro, puede ser necesario migrar las pruebas a una base de datos más cercana a producción.
