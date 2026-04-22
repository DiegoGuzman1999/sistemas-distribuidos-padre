# ADR-001 — Arquitectura de Microservicios en lugar de Monolito

**Fecha:** 2026-04-07
**Estado:** Aceptado
**Autores:** DiegoGuzman1999, Checho999awoo

---

## Contexto

El sistema de inventario requería gestionar dos responsabilidades distintas: autenticación de usuarios y gestión de productos/ventas. Era necesario decidir si construir una única aplicación que manejara todo (monolito) o dividirla en servicios independientes.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| Monolito | Una sola aplicación Flask con todos los endpoints |
| Microservicios | Dos backends Flask independientes, cada uno con su responsabilidad |

## Decisión

Se eligió **arquitectura de microservicios** con dos backends separados:
- `backend-1`: autenticación (login, logout, verificar sesión)
- `backend-2`: inventario y ventas (CRUD de productos, registro de ventas)

## Justificación

- Si el servicio de autenticación falla, el inventario puede seguir operando de forma independiente.
- Cada servicio puede escalar, desplegarse y probarse de forma autónoma.
- Cumple directamente con el objetivo del curso de Sistemas Distribuidos.
- Permite que dos desarrolladores trabajen en paralelo sin conflictos de código.

## Consecuencias

- **Positivas:** Independencia entre servicios, separación de responsabilidades, facilidad para pruebas unitarias por servicio.
- **Negativas:** Mayor complejidad inicial al configurar dos contenedores y coordinar la comunicación entre ellos.
