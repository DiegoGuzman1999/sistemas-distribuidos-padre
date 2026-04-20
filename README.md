# sistemas-distribuidos-padre

## ¿Qué es este proyecto?

Sistema de inventario web con login, construido con arquitectura de **microservicios** como proyecto de Sistemas Distribuidos. Cada funcionalidad vive en su propio servicio independiente, todos orquestados desde este repositorio.

---

## ¿Por qué microservicios?

En lugar de tener un solo programa que hace todo, el sistema se divide en piezas independientes:

- Si el servicio de inventario falla, el login sigue funcionando
- Cada servicio puede actualizarse sin afectar a los demás
- Representa cómo funcionan sistemas reales en producción (Netflix, Amazon, etc.)

---

## Repositorios del proyecto

| Repositorio | Función | Puerto |
|---|---|---|
| **sistemas-distribuidos-padre** | Orquesta todo con Docker Compose | — |
| **sistemas-distribuidos-backend-1** | API de autenticación (login/logout) | 5000 |
| **sistemas-distribuidos-backend-2** | API de inventario (CRUD productos) | 5001 |
| **sistemas-distribuidos-frontend** | Interfaz web HTML/CSS | 3000 |
| **sistemas-distribuidos-bd** | Base de datos PostgreSQL + Liquibase | 5432 |

---

## Stack tecnológico

| Capa | Tecnología | ¿Para qué? |
|---|---|---|
| Backend | Python + Flask | APIs REST de login e inventario |
| Base de datos | PostgreSQL | Almacena usuarios y productos |
| Migraciones | Liquibase | Crea las tablas automáticamente al iniciar |
| Contenedores | Docker + Docker Compose | Empaqueta y levanta todos los servicios |
| Frontend | HTML + CSS + JavaScript | Interfaz que consume las APIs |

---

## Requisitos previos

- Tener instalado **Docker Desktop** — [Descargar aquí](https://www.docker.com/products/docker-desktop/)
- Tener clonados los 5 repositorios en la misma carpeta

### Estructura de carpetas esperada
```
mi-carpeta/
├── repositorio1/   ← sistemas-distribuidos-padre  (este repo)
├── repositorio2/   ← sistemas-distribuidos-backend-1
├── repositorio3/   ← sistemas-distribuidos-backend-2
├── repositorio4/   ← sistemas-distribuidos-frontend
└── repositorio5/   ← sistemas-distribuidos-bd
```

---

## ¿Cómo correr el proyecto?

### 1. Abrir una terminal dentro de esta carpeta (repositorio1)

### 2. Levantar todos los servicios
```bash
docker compose up --build
```
> La primera vez tarda varios minutos porque descarga las imágenes. Las siguientes veces es más rápido.

### 3. Abrir el navegador en
```
http://localhost:3000
```

### 4. Ingresar con las credenciales iniciales
- **Usuario:** `admin`
- **Contraseña:** `admin123`

### 5. Apagar el sistema cuando termines
```bash
docker compose down
```

---

## Orden de arranque de los servicios

Docker Compose levanta los servicios en este orden automáticamente:

```
1. PostgreSQL        ← Se levanta primero y espera hasta estar listo
2. Liquibase         ← Crea las tablas y el usuario admin (solo una vez)
3. Backend-1 y 2     ← Las APIs Flask arrancan después de la BD
4. Frontend          ← La interfaz web arranca de última
```

---

## Flujo de ramas

```
dev  →  qa  →  main
```

| Rama | Propósito |
|---|---|
| `dev` | Desarrollo activo |
| `qa` | Pruebas antes de producción |
| `main` | Versión estable y probada |

---

## Credenciales de la base de datos

| Campo | Valor |
|---|---|
| Host | localhost |
| Puerto | 5432 |
| Base de datos | inventario_db |
| Usuario | admin |
| Contraseña | admin123 |

---

## Comandos útiles

| Quiero... | Comando |
|---|---|
| Levantar con cambios de código | `docker compose up --build` |
| Levantar sin cambios | `docker compose up` |
| Apagar todo | `docker compose down` |
| Apagar y borrar datos de BD | `docker compose down -v` |

---

*Proyecto de Sistemas Distribuidos 2026 — Diego Guzman*
