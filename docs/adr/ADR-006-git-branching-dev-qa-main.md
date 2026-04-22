# ADR-006 — Estrategia de Ramas Git: dev → qa → main

**Fecha:** 2026-04-20
**Estado:** Aceptado
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El proyecto involucra dos desarrolladores trabajando sobre 5 repositorios. Era necesario definir una estrategia de control de versiones que permitiera desarrollar nuevas funcionalidades, probarlas y promoverlas a producción de forma ordenada y sin romper el código estable.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Rama única (`main`) | Todo el trabajo directo sobre main |
| Feature branches | Una rama por funcionalidad, merge a main al terminar |
| Flujo `dev → qa → main` | Tres ramas fijas que representan el estado del sistema en cada etapa |

## Decisión

Se adoptó el flujo de **tres ramas fijas** con promoción progresiva:

```
dev → qa → main
```

| Rama | Propósito |
|---|---|
| `dev` | Desarrollo activo — aquí se sube código nuevo |
| `qa` | Pruebas — código que funciona en dev se promueve aquí |
| `main` | Producción — código estable y completamente probado |

Cada rama tiene su propio ambiente Docker dedicado (ver ADR-004), lo que hace que el estado del código y el estado del sistema estén siempre sincronizados.

## Justificación

- Refleja el flujo real de un equipo de desarrollo profesional con ambientes separados.
- Protege la rama `main` de cambios no probados.
- Permite que dos desarrolladores trabajen en `dev` en paralelo sin afectar los ambientes de QA y producción.
- Se alinea directamente con los requisitos del proyecto de Sistemas Distribuidos (15 ambientes: 5 servicios × 3 ramas).
- La promoción `git merge` entre ramas es explícita y trazable en el historial de Git.

## Consecuencias

- **Positivas:** Código siempre estable en `main`, trazabilidad completa del flujo de cambios, alineación entre ramas y ambientes Docker.
- **Negativas:** Requiere disciplina del equipo para no saltarse etapas. Los cambios deben mergearse de `dev` a `qa` y de `qa` a `main` manualmente, lo que puede generar conflictos si las ramas divergen por mucho tiempo.
