# AVANCE — sistemas-distribuidos-padre

## ¿Qué hace este repositorio?
Es el **orquestador** del proyecto. Contiene el `docker-compose.yml` principal que levanta todos los servicios juntos con un solo comando. No tiene código propio — su función es conectar y coordinar los demás repositorios.

---

## ¿Qué es Docker Compose?
Es una herramienta que permite definir y correr múltiples contenedores Docker al mismo tiempo. En lugar de levantar cada servicio manualmente, con un solo comando (`docker compose up`) se levanta todo el sistema.

---

## Arquitectura multi-ambiente

Cada rama tiene su propio ambiente completamente independiente con red, base de datos y puertos separados:

```
┌─────────────────── DEV ───────────────────┐
│  [Frontend :3000] → /auth/ → [Auth :5000] │
│                  → /inv/  → [Inv  :5001]  │
│  [Auth] [Inv] → [PostgreSQL :5432]        │
└───────────────────────────────────────────┘

┌─────────────────── QA ────────────────────┐
│  [Frontend :3001] → /auth/ → [Auth :5010] │
│                  → /inv/  → [Inv  :5011]  │
│  [Auth] [Inv] → [PostgreSQL :5433]        │
└───────────────────────────────────────────┘

┌─────────────────── MAIN ──────────────────┐
│  [Frontend :3002] → /auth/ → [Auth :5020] │
│                  → /inv/  → [Inv  :5021]  │
│  [Auth] [Inv] → [PostgreSQL :5434]        │
└───────────────────────────────────────────┘
```

---

## Puertos por ambiente

| Servicio | Dev | QA | Main |
|---|---|---|---|
| Frontend | 3000 | 3001 | 3002 |
| Auth API | 5000 | 5010 | 5020 |
| Inventario API | 5001 | 5011 | 5021 |
| PostgreSQL | 5432 | 5433 | 5434 |

---

## Estructura del repositorio

```
sistemas-distribuidos-padre/
├── nginx/
│   ├── nginx.dev.conf    ← Proxy Nginx para ambiente dev
│   ├── nginx.qa.conf     ← Proxy Nginx para ambiente qa
│   └── nginx.main.conf   ← Proxy Nginx para ambiente main
├── docker-compose.dev.yml  ← Levanta los 5 servicios en dev
├── docker-compose.qa.yml   ← Levanta los 5 servicios en qa
├── docker-compose.main.yml ← Levanta los 5 servicios en main
├── docker-compose.yml      ← Compose general (legacy)
├── README.md
└── AVANCE.md
```

---

## Orden de arranque (igual en los 3 ambientes)
1. **PostgreSQL** — espera hasta estar listo
2. **Liquibase** — crea las tablas y el usuario admin (solo una vez)
3. **Backend-1** y **Backend-2** — APIs Flask
4. **Frontend** — Nginx con proxy hacia los backends

---

## Comandos por ambiente

```bash
# DEV
docker compose -f docker-compose.dev.yml up --build

# QA
docker compose -f docker-compose.qa.yml up --build

# MAIN
docker compose -f docker-compose.main.yml up --build

# Apagar un ambiente
docker compose -f docker-compose.dev.yml down
```

## URLs por ambiente

| Ambiente | URL |
|---|---|
| DEV | http://localhost:3000 |
| QA | http://localhost:3001 |
| MAIN | http://localhost:3002 |

---

## Independencia de contenedores
Cada contenedor tiene `restart: unless-stopped` — si uno cae, Docker lo reinicia automáticamente sin afectar los otros ambientes. Cada ambiente usa su propia red Docker (`red-dev`, `red-qa`, `red-main`).

---

## Credenciales iniciales
- **Usuario:** admin
- **Contraseña:** admin123

---

## Historial de cambios

| Fecha      | Autor | Rama | Descripción |
|------------|-------|------|-------------|
| 2026-04-20 | DiegoGuzman1999 | dev | Docker Compose general que orquesta todos los servicios |
| 2026-04-21 | DiegoGuzman1999 | dev | Arquitectura multi-ambiente: docker-compose por dev/qa/main con puertos y redes independientes |
