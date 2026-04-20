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

## Historias de Usuario

### HU-001 — Iniciar Sesión
**Como** usuario administrador,
**quiero** iniciar sesión con mi usuario y contraseña,
**para** acceder al sistema de inventario de forma segura.

**Criterios de aceptación:**
- El sistema valida que el usuario exista en la base de datos
- Si las credenciales son incorrectas, muestra un mensaje de error
- Si son correctas, redirige al dashboard de inventario
- La sesión persiste mientras el usuario no cierre sesión

---

### HU-002 — Cerrar Sesión
**Como** usuario administrador autenticado,
**quiero** cerrar mi sesión,
**para** proteger el acceso al sistema cuando termine de usarlo.

**Criterios de aceptación:**
- El botón de logout está visible en todas las páginas
- Al cerrar sesión, redirige al formulario de login
- No se puede acceder al inventario sin estar autenticado

---

### HU-003 — Ver Inventario
**Como** usuario administrador,
**quiero** ver la lista completa de productos del inventario,
**para** conocer el stock disponible.

**Criterios de aceptación:**
- Se muestra una tabla con todos los productos
- La tabla incluye: nombre, descripción, cantidad y precio
- Solo usuarios autenticados pueden ver el inventario

---

### HU-004 — Agregar Producto
**Como** usuario administrador,
**quiero** agregar nuevos productos al inventario,
**para** mantener actualizado el stock.

**Criterios de aceptación:**
- Existe un formulario para ingresar nombre, descripción, cantidad y precio
- Todos los campos obligatorios deben estar completos
- Al guardar, el producto aparece en la lista de inventario
- Se muestra un mensaje de éxito al agregar correctamente

---

### HU-005 — Editar Producto
**Como** usuario administrador,
**quiero** editar la información de un producto existente,
**para** corregir o actualizar datos del inventario.

**Criterios de aceptación:**
- Cada producto tiene un botón de editar
- El formulario se precarga con los datos actuales del producto
- Al guardar, los cambios se reflejan inmediatamente en la lista
- Se muestra un mensaje de éxito al editar correctamente

---

### HU-006 — Eliminar Producto
**Como** usuario administrador,
**quiero** eliminar un producto del inventario,
**para** remover artículos que ya no están disponibles.

**Criterios de aceptación:**
- Cada producto tiene un botón de eliminar
- Aparece una confirmación antes de eliminar
- Al confirmar, el producto desaparece de la lista
- Se muestra un mensaje de éxito al eliminar correctamente

---

### Resumen de H.U.

| ID | Historia | Prioridad | Estimación | Servicio |
|---|---|---|---|---|
| HU-001 | Iniciar Sesión | Alta | 3 puntos | backend-1 |
| HU-002 | Cerrar Sesión | Alta | 1 punto | backend-1 |
| HU-003 | Ver Inventario | Alta | 2 puntos | backend-2 |
| HU-004 | Agregar Producto | Media | 3 puntos | backend-2 |
| HU-005 | Editar Producto | Media | 3 puntos | backend-2 |
| HU-006 | Eliminar Producto | Media | 2 puntos | backend-2 |

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
