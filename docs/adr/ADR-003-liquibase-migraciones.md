# ADR-003 — Usar Liquibase para Migraciones de Base de Datos

**Fecha:** 2026-04-07
**Estado:** Aceptado
**Autores:** DiegoGuzman1999

---

## Contexto

Al levantar el sistema con Docker, la base de datos PostgreSQL debía inicializarse con las tablas y datos necesarios. Era necesario decidir cómo gestionar la creación del esquema y los datos iniciales de forma reproducible en los 3 ambientes.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Scripts SQL manuales | Ejecutar `psql` manualmente cada vez que se levanta el sistema |
| Script de inicialización en PostgreSQL | Usar el directorio `docker-entrypoint-initdb.d` de la imagen oficial |
| Liquibase | Herramienta de control de versiones para bases de datos |

## Decisión

Se eligió **Liquibase** con un contenedor dedicado que corre antes que los backends.

```xml
<databaseChangeLog>
    <include file="changelog/001-crear-tablas.sql"/>
    <include file="changelog/002-datos-iniciales.sql"/>
    <include file="changelog/003-tabla-ventas.sql"/>
</databaseChangeLog>
```

## Justificación

- Liquibase lleva un registro interno (`DATABASECHANGELOG`) de qué scripts ya fueron ejecutados, evitando duplicados al reiniciar el sistema.
- Agregar nuevas tablas o columnas en el futuro solo requiere añadir un nuevo script numerado, sin tocar los anteriores.
- Es reproducible: los 3 ambientes (DEV, QA, MAIN) siempre quedan con el mismo esquema.
- El contenedor de Liquibase usa `depends_on` con `condition: service_healthy`, garantizando que PostgreSQL esté listo antes de migrar.

## Consecuencias

- **Positivas:** Migraciones versionadas, reproducibles y auditables. Facilita agregar cambios al esquema en el futuro.
- **Negativas:** Agrega un contenedor adicional al stack y requiere mantener el archivo `changelog-master.xml` actualizado con cada nuevo script.
