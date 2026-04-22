# Guía de Pruebas por Ambiente

## Requisitos previos
- Tener Docker Desktop instalado y corriendo
- Tener los 5 repositorios clonados en la misma carpeta
- Estar ubicado en la carpeta `sistemas-distribuidos-padre`

---

## Estructura de ambientes

| Ambiente | Rama | Frontend | Auth API | Inventario API | Base de Datos |
|---|---|---|---|---|---|
| DEV | `dev` | http://localhost:3000 | :5000 | :5001 | :5432 |
| QA | `qa` | http://localhost:3001 | :5010 | :5011 | :5433 |
| MAIN | `main` | http://localhost:3002 | :5020 | :5021 | :5434 |

---

## Levantar los ambientes

### Solo DEV
```bash
docker compose -f docker-compose.dev.yml up --build
```

### Solo QA
```bash
docker compose -f docker-compose.qa.yml up --build
```

### Solo MAIN
```bash
docker compose -f docker-compose.main.yml up --build
```

### Los 3 al mismo tiempo
Abrir 3 terminales distintas en la misma carpeta y correr un comando en cada una.

---

## Verificar que todo está corriendo

```bash
docker ps
```

Deben aparecer 12 contenedores activos:

```
frontend-dev        → puerto 3000
backend-autenticacion-dev   → puerto 5000
backend-inventario-dev      → puerto 5001
postgres-dev        → puerto 5432

frontend-qa         → puerto 3001
backend-autenticacion-qa    → puerto 5010
backend-inventario-qa       → puerto 5011
postgres-qa         → puerto 5433

frontend-main       → puerto 3002
backend-autenticacion-main  → puerto 5020
backend-inventario-main     → puerto 5021
postgres-main       → puerto 5434
```

---

## Pruebas funcionales

Repetir los siguientes pasos en cada ambiente cambiando la URL correspondiente.

---

### Prueba 1 — Login exitoso

**URL:** `http://localhost:3000` (DEV) / `3001` (QA) / `3002` (MAIN)

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Abrir la URL en el navegador | Aparece el formulario de login |
| 2 | Ingresar usuario: `admin` y contraseña: `admin123` | Redirige al inventario |
| 3 | Verificar que aparece `Hola, admin` en la barra superior | Sesión iniciada correctamente |

---

### Prueba 2 — Login fallido

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Ingresar usuario: `admin` y contraseña: `incorrecta` | Aparece mensaje de error en rojo |
| 2 | Verificar que NO redirige al inventario | Se queda en la pantalla de login |

---

### Prueba 3 — Ver inventario

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Iniciar sesión correctamente | Aparece la tabla de productos |
| 2 | Si no hay productos | Mensaje "No hay productos en el inventario" |
| 3 | Verificar que la tabla tiene columnas: ID, Nombre, Descripción, Cantidad, Precio | Tabla visible |

---

### Prueba 4 — Agregar producto

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Llenar el formulario con: Nombre: `Laptop`, Cantidad: `10`, Precio: `1500` | — |
| 2 | Clic en **Guardar** | Mensaje de éxito en verde |
| 3 | Verificar que el producto aparece en la tabla | Producto agregado |

---

### Prueba 5 — Editar producto

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Clic en **Editar** en cualquier producto | Formulario se precarga con los datos actuales |
| 2 | Cambiar la cantidad a `20` | — |
| 3 | Clic en **Guardar** | Mensaje de éxito, tabla actualizada |

---

### Prueba 6 — Eliminar producto

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Clic en **Eliminar** en cualquier producto | Aparece ventana de confirmación |
| 2 | Confirmar la eliminación | Producto desaparece de la tabla |

---

### Prueba 7 — Histórico de ventas

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Clic en **Ver Histórico de Ventas** | Abre la página de ventas |
| 2 | Seleccionar un producto y cantidad `2` | — |
| 3 | Clic en **Registrar Venta** | Mensaje de éxito |
| 4 | Verificar que el stock del producto bajó en 2 | Stock actualizado |
| 5 | Verificar que aparece en el historial | Venta registrada con fecha |
| 6 | Verificar las tarjetas de resumen | Total ventas, ingresos y productos actualizados |

---

### Prueba 8 — Cerrar sesión

| Paso | Acción | Resultado esperado |
|---|---|---|
| 1 | Clic en **Cerrar Sesión** | Redirige al login |
| 2 | Intentar ir directamente a `/inventario.html` | Redirige al login automáticamente |

---

## Prueba de independencia entre ambientes

Esta prueba verifica que si un contenedor falla, los demás siguen funcionando.

### Paso a paso

**1. Levantar los 3 ambientes** y verificar que los 3 funcionan en el navegador.

**2. Agregar un producto en DEV** (`http://localhost:3000`) con nombre `Producto DEV`.

**3. Verificar que NO aparece en QA** (`http://localhost:3001`).
Esto confirma que cada ambiente tiene su propia base de datos.

**4. Apagar un contenedor de QA:**
```bash
docker stop frontend-qa
```

**5. Verificar que:**
- `http://localhost:3001` → ya no responde (QA caído)
- `http://localhost:3000` → sigue funcionando (DEV intacto)
- `http://localhost:3002` → sigue funcionando (MAIN intacto)

**6. Restaurar el contenedor:**
```bash
docker start frontend-qa
```

**7. Verificar que QA vuelve a responder** en `http://localhost:3001`.

---

## Prueba de reinicio automático

```bash
# Matar el proceso del backend de DEV de forma forzada
docker kill backend-autenticacion-dev

# Esperar 5 segundos y verificar que Docker lo reinició solo
docker ps | grep backend-autenticacion-dev
```

El contenedor debe aparecer con estado `Up X seconds` — Docker lo reinició automáticamente gracias a `restart: unless-stopped`.

---

## Ver logs de un servicio

Si algo falla, ver los logs del contenedor:

```bash
# Logs del backend de autenticacion en DEV
docker logs backend-autenticacion-dev

# Logs del backend de inventario en QA
docker logs backend-inventario-qa

# Logs del frontend en MAIN
docker logs frontend-main

# Seguir los logs en tiempo real (Ctrl+C para salir)
docker logs -f backend-autenticacion-dev
```

---

## Apagar ambientes

```bash
# Apagar DEV
docker compose -f docker-compose.dev.yml down

# Apagar QA
docker compose -f docker-compose.qa.yml down

# Apagar MAIN
docker compose -f docker-compose.main.yml down

# Apagar todo y borrar datos de BD
docker compose -f docker-compose.dev.yml down -v
docker compose -f docker-compose.qa.yml down -v
docker compose -f docker-compose.main.yml down -v
```

---

*Guía de pruebas — Proyecto Sistemas Distribuidos 2026*
