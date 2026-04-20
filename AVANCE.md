# AVANCE — sistemas-distribuidos-padre

## ¿Qué hace este repositorio?
Es el **orquestador** del proyecto. Contiene el `docker-compose.yml` principal que levanta todos los servicios juntos con un solo comando. No tiene código propio — su función es conectar y coordinar los demás repositorios.

---

## ¿Qué es Docker Compose?
Es una herramienta que permite definir y correr múltiples contenedores Docker al mismo tiempo. En lugar de levantar cada servicio manualmente, con un solo comando (`docker compose up`) se levanta todo el sistema.

---

## Arquitectura del sistema

```
┌─────────────────────────────────────────────────┐
│                  red-inventario                 │
│                                                 │
│  [Frontend :3000] ──→ [Auth API  :5000]         │
│                   ──→ [Inv. API  :5001]         │
│                                                 │
│  [Auth API]  ──→ [PostgreSQL :5432]             │
│  [Inv. API]  ──→ [PostgreSQL :5432]             │
│  [Liquibase] ──→ [PostgreSQL] (solo al inicio)  │
└─────────────────────────────────────────────────┘
```

---

## Orden de arranque
Docker Compose levanta los servicios en este orden:
1. **PostgreSQL** — la base de datos (espera hasta estar lista)
2. **Liquibase** — crea las tablas y el usuario admin (solo se ejecuta una vez)
3. **Backend-1** y **Backend-2** — las APIs Flask
4. **Frontend** — la interfaz web

---

## Cómo correr el proyecto completo

```bash
# Clonar todos los repos en la misma carpeta
# Luego desde esta carpeta (padre):
docker compose up --build

# Para apagar todo:
docker compose down

# Para apagar y borrar los datos de la BD:
docker compose down -v
```

## URLs del sistema (una vez levantado)

| Servicio        | URL                    |
|-----------------|------------------------|
| Frontend        | http://localhost:3000  |
| API Auth        | http://localhost:5000  |
| API Inventario  | http://localhost:5001  |
| PostgreSQL      | localhost:5432         |

---

## Credenciales iniciales
- **Usuario:** admin
- **Contraseña:** admin123

---

## Historial de cambios

| Fecha      | Rama | Descripción                                        |
|------------|------|----------------------------------------------------|
| 2026-04-20 | dev  | Docker Compose general que orquesta todos los servicios |
