# ADR-002 — Usar Nginx como Proxy Inverso en lugar de URLs hardcodeadas

**Fecha:** 2026-04-07
**Estado:** Aceptado
**Autores:** DiegoGuzman1999

---

## Contexto

El frontend necesitaba comunicarse con los dos backends. La primera implementación usaba URLs absolutas hardcodeadas (`http://localhost:5000`, `http://localhost:5001`). Esto hacía que el mismo HTML no funcionara en los ambientes QA y MAIN, que usan puertos distintos.

## Opciones evaluadas

| Opción | Descripción |
|---|---|
| URLs hardcodeadas por ambiente | Un archivo HTML distinto por ambiente con sus puertos |
| Variable de entorno en JS | Inyectar la URL del backend al momento de construir |
| Nginx como proxy inverso | El frontend usa rutas relativas y Nginx redirige al backend correcto |

## Decisión

Se eligió **Nginx como proxy inverso**. El HTML hace llamadas a rutas relativas (`/auth/`, `/inv/`) y Nginx las redirige al backend correspondiente dentro de la red Docker.

```nginx
location /auth/ { proxy_pass http://backend-autenticacion-dev:5000/; }
location /inv/  { proxy_pass http://backend-inventario-dev:5001/; }
```

## Justificación

- El mismo HTML funciona en los 3 ambientes sin ningún cambio de código.
- La configuración de Nginx se inyecta desde el repo padre vía volumen Docker, manteniendo la separación de responsabilidades.
- Nginx actúa como punto de entrada único, simplificando la gestión de CORS.
- Es la práctica estándar en sistemas distribuidos con frontend estático.

## Consecuencias

- **Positivas:** Un solo frontend sirve para DEV, QA y MAIN. Centralización del enrutamiento.
- **Negativas:** Se requiere un archivo `nginx.conf` por ambiente (`nginx.dev.conf`, `nginx.qa.conf`, `nginx.main.conf`), lo que agrega archivos de configuración a mantener.
