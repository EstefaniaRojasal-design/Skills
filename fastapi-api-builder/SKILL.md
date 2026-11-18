---
name: fastapi-api-builder
description: Guía de principio a fin para diseñar, construir, probar, empaquetar y desplegar APIs REST con Python y FastAPI. Úsalo siempre que el usuario quiera crear, diseñar, construir, exponer o desplegar una API, backend, servicio web, endpoint, webhook, o convertir un script/proceso en un servicio invocable por bots RPA, Automation Anywhere, apps o frontends, aunque no mencione FastAPI ni despliegue.
---

# Constructor de APIs con FastAPI: del diseño al despliegue

Guía al usuario por seis fases: entender → diseñar → construir → probar → empaquetar → desplegar. Pregunta solo lo que cambie el resultado, asume el resto declarando las suposiciones y construye sin solicitar aprobación fase por fase.

## Responsabilidades y límites

- Genera y prueba el código en el entorno disponible.
- El despliegue real lo ejecuta el usuario en su cuenta o servidor, porque el asistente normalmente no tiene acceso. Entrega comandos exactos y pasos de verificación.
- Si el entorno tiene acceso al servidor o CLI de plataforma, por ejemplo Claude Code, ejecuta las acciones solo con la confirmación necesaria del usuario.
- No afirmes que ejecutaste pruebas o despliegues si no se pudieron realizar.

## Fase 1: Entender (una ronda de preguntas clave)

Antes de escribir código, aclara lo que cambia la arquitectura. Si hay una herramienta de preguntas con opciones tocables (`ask_user_input_v0`), úsala; si no, haz una lista corta. Máximo tres o cuatro preguntas y omite las ya respondidas:

1. **Propósito y recursos:** ¿qué hace la API, sobre qué entidades opera (por ejemplo clientes, facturas, órdenes) y qué operaciones necesita?
2. **Persistencia:** ¿sin base de datos, SQLite (prototipo) o PostgreSQL?
3. **Consumidor y acceso:** ¿quién la llama (bot RPA/Automation Anywhere, app web, otro servicio) y debe ser pública o interna? Esto determina autenticación y despliegue.
4. **Destino:** si ya se conoce, úsalo; si no, presenta opciones de la Fase 6 y recomienda una sin bloquear la construcción.

Para lo demás, asumir valores razonables (FastAPI, API key, formato uniforme de errores, pytest, Docker) y declararlos en una línea.

## Fase 2: Diseñar

Antes de construir, presenta un diseño compacto para que el usuario pueda corregirlo; si confirma o no objeta, continúa a construir:

- Tabla de endpoints: método, ruta, función y códigos de respuesta.
- Modelos de entrada/salida: campos y validaciones.
- Autenticación seleccionada y justificación; consultar `references/auth.md` si existe.

Convenciones REST: sustantivos plurales (`/clientes`), verbos HTTP para la acción, 201 al crear, 204 al borrar, 404 si no existe, 422 para datos inválidos y 401 sin credenciales.

## Fase 3: Construir

Si están disponibles, copia la plantilla probada mediante el scaffold en lugar de reescribirla:

```bash
python <ruta-del-skill>/scripts/scaffold.py "<nombre-del-proyecto>" --out /home/claude
```

Adapta la plantilla al diseño:
- Reemplaza el recurso de ejemplo `app/routers/items.py` y `app/schemas.py` por los recursos requeridos.
- Registra routers en `app/main.py` con `include_router`.
- Ajusta `app/config.py` y `.env.example` para variables nuevas.
- Si hay base de datos, consulta `references/database.md` si existe.
- Si necesita login de usuarios, consulta `references/auth.md` si existe.
- Si el consumidor es Automation Anywhere, consulta `references/aa-integration.md` si existe y aplica su contrato: códigos HTTP claros, JSON con `status` y mensajes útiles.

No inventes archivos de plantilla ni referencias que no estén disponibles. Si no existen, crea una solución equivalente explícitamente.

La estructura por capas (routers, schemas, config, security) facilita el crecimiento; Pydantic valida datos antes de la lógica; manejadores globales devuelven errores uniformes como `{"status":"error","message":"..."}` sin filtrar detalles internos en errores 500. Incluir `/health` para Docker, balanceadores y plataformas. Configurar por variables de entorno: mismo código local y en producción, secretos fuera del repositorio.

## Fase 4: Probar

Ejecuta siempre antes de entregar y corrige fallos hasta que pasen, cuando el entorno lo permita:

```bash
cd <proyecto> && pip install -r requirements-dev.txt && python -m pytest -q
```

Agregar pruebas para cada endpoint nuevo: caso feliz, dato inválido, recurso no encontrado y sin autenticación cuando aplique. Si es posible, iniciar el servidor real (`uvicorn app.main:app --port 8000`) y verificar con `curl` `/health`, endpoint protegido con y sin clave, y `/docs`. Si alguna prueba no puede ejecutarse, explicar por qué y qué falta.

## Fase 5: Empaquetar

- El Dockerfile debe correr como usuario sin privilegios y escuchar en `$PORT`, para funcionar igual en Render, Railway, Cloud Run y VPS.
- `.dockerignore` y `.gitignore` deben excluir `.env`; comprobar que no haya secretos en el código ni en los archivos entregados.
- Entregar proyecto completo comprimido en `/mnt/user-data/outputs/` con `present_files`, si esas herramientas están disponibles. Si no, proporcionar estructura y archivos en forma útil, sin afirmar que se creó un ZIP.
- Incluir resumen breve: cómo correr localmente, variables de entorno necesarias y endpoints.

### Lista previa a producción

- [ ] `API_KEY` (o JWT) definida con valor largo y aleatorio, guardada como secreto en la plataforma.
- [ ] `CORS_ORIGINS` con lista explícita, no `*`.
- [ ] `ENVIRONMENT=production`.
- [ ] Base de datos persistente y con backups; no usar SQLite en disco efímero.
- [ ] HTTPS activo; nunca exponer una API key por HTTP.
- [ ] `.env` fuera del repositorio.
- [ ] Pruebas pasando y `/health` respondiendo.
- [ ] Logs accesibles y alerta ante errores 5xx.

## Fase 6: Desplegar

Si el usuario no eligió destino, presentar esta comparación y recomendar según su caso. Si tiene poca experiencia en infraestructura, PaaS; si requiere red interna o control total, VPS/servidor propio; si la empresa ya usa una nube, contenedor gestionado en esa nube.

| Opción | Esfuerzo | Control | Costo típico | Mejor para |
|---|---|---|---|---|
| Docker en VPS o servidor propio | Medio | Total | Fijo y bajo | Red interna, control, costo predecible |
| PaaS (Render, Railway, Fly.io) | Bajo | Medio | Bajo a medio, por uso | Salir rápido, equipos pequeños |
| Nube (Cloud Run, Container Apps, App Runner/ECS) | Medio a alto | Alto | Por uso | Empresas con nube, escalado, cumplimiento |

Después de elegir, consulta solo la guía correspondiente, si existe:
- `references/deploy-vps-docker.md`
- `references/deploy-paas.md`
- `references/deploy-cloud.md`

Los comandos y pantallas de plataformas externas cambian. Si el usuario indica que la interfaz no coincide, consulta documentación vigente en vez de insistir con instrucciones desactualizadas.

Cierra con verificación posterior al despliegue:

```bash
curl https://<url>/health
```

Abrir `/docs`, probar un endpoint con y sin clave (sin ella debe responder 401) y revisar logs. Si falla, pedir error o logs y diagnosticar. Problemas comunes: puerto incorrecto (la app debe escuchar en `$PORT`), variable de entorno faltante, `postgres://` en lugar de `postgresql://`, o CORS bloqueando al frontend.

## Formato de entrega final

Mantenerlo breve y útil e incluir:
1. Qué se construyó y suposiciones tomadas.
2. Comandos para correrlo localmente.
3. Variables de entorno requeridas.
4. Tabla de endpoints.
5. Siguientes pasos del despliegue elegido.

Evitar repetir innecesariamente el contenido de los archivos.
