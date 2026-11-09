# Catálogo de Estándares para Validación Automática de PR

Este catálogo es **configurable por el equipo/organización** (ajusta categorías, criterios, pesos y umbrales para que coincida con la política real de tu banco/empresa). El agente `standards-compliance-validator` lee este archivo para saber qué evaluar y cómo puntuar cada categoría.

## Regla de bloqueo global

- **BLOQUEAR PR: SI** en una categoría si: existe al menos 1 hallazgo `CRÍTICO`, **O** el score de esa categoría es `< 7/10`.
- **BLOQUEAR PR: NO** en caso contrario.
- **Decisión Global de Bloqueo = SI** si CUALQUIER categoría individual bloquea. Si ninguna bloquea, Global = NO.
- Una categoría sin archivos relevantes en el diff se marca `NO APLICA` (Score 10/10, no bloquea) — nunca se "adivina" un hallazgo sin evidencia en el diff.

## Escala de criticidad de hallazgos

| Criticidad | Significado | Efecto en score |
|---|---|---|
| 🔴 CRÍTICO | Bloquea el PR por sí solo (seguridad, secretos expuestos, breaking change sin mitigar, perdida de datos) | Fuerza bloqueo de la categoría |
| 🟡 MEDIO | Debe corregirse pronto, no bloquea por sí solo | -2 pts |
| 🔵 BAJO | Mejora recomendada | -1 pt |
| ℹ️ NO APLICA / INFO | Informativo, sin impacto | 0 pts |

Cada categoría parte de 10 puntos y resta segun los hallazgos encontrados (score mínimo 0).

---

## 1. Estándares de Pruebas Unitarias

**Aplica si:** el diff toca código de producción (no solo config/build/docs).

**Criterios:**
- Todo archivo de lógica de negocio modificado/agregado tiene un test correspondiente modificado/agregado (3 pts).
- Los tests existentes que cubren el área modificada siguen siendo válidos (no quedaron desactualizados respecto al nuevo comportamiento) (3 pts).
- Nuevas ramas de código (if/else, manejo de errores) tienen al menos un caso de test (2 pts).
- No hay tests eliminados sin justificación ni marcados `@Disabled`/`skip` sin ticket asociado (2 pts).

## 2. Estándares de Arquitectura

**Aplica si:** el diff agrega/modifica código fuente, no solo dependencias/config.

**Criterios:**
- Se respeta la separación de capas del proyecto (HTTP/controller no contiene lógica de negocio pesada, no accede directo a infraestructura) (4 pts).
- No se introduce acoplamiento nuevo innecesario (un módulo importando directamente detalles internos de otro en vez de una interfaz/contrato) (3 pts).
- Nombres, paquetes/carpetas y convenciones siguen el patrón ya usado en el resto del repo (3 pts).

## 3. Estándares de Seguridad (OWASP)

**Aplica siempre que haya cambios en código fuente, configuración, o contratos de API (OpenAPI/GraphQL/proto).**

**Criterios:**
- Validación server-side de toda entrada externa, no solo confiar en el contrato/schema (2 pts).
- Estrategia allowlist para campos con valores/patrones restringidos, aplicada en backend (no solo documentada) (3 pts).
- Manejo seguro de secretos/credenciales (sin hardcodear, sin loguearlos) (3 pts, ver también categoría de Resiliencia Config — un hallazgo aquí se duplica ahí si aplica).
- Sin expresiones regulares vulnerables a ReDoS, sin construcción de queries/comandos por concatenación de input no sanitizado (2 pts).

## 4. Resiliencia Nivel 1 — Configuración de la Aplicación

**Aplica si:** el diff toca archivos de configuración (`.yml`, `.env`, `.properties`, Helm/K8s, `.tfvars`, etc.).

**Criterios:**
- Sin credenciales/secretos hardcodeados en ningún archivo de config, ni siquiera en perfiles "local"/"dev" (4 pts, cualquier violación = hallazgo CRÍTICO automático).
- Toda configuración sensible o dependiente de ambiente está externalizada (variables de entorno / secret manager), no hardcodeada por ambiente (3 pts).
- Nomenclatura de variables de configuración sigue el estándar del equipo (3 pts).

## 5. Resiliencia Nivel 1 — Gestión de Logs

**Aplica si:** el diff toca código fuente o configuración de logging.

**Criterios:**
- No se loguea información sensible (passwords, tokens, PII sin enmascarar) (4 pts, violación = CRÍTICO).
- Logs estructurados (formato consistente, JSON o el usado en el repo) con nivel apropiado (INFO/WARN/ERROR) para eventos y excepciones relevantes (3 pts).
- Nivel de log configurable externamente (no hardcodeado) (3 pts).

## 6. Resiliencia Nivel 2 — Health Checks y Timeouts

**Aplica si:** el diff toca llamadas a servicios externos/DB, configuración de Actuator/health, o clientes HTTP.

**Criterios:**
- Endpoints/health checks de liveness y readiness definidos y expuestos de forma segura (`show-details: never` o equivalente) (5 pts).
- Timeouts explícitos (connect/read) configurados y realmente usados por los clientes HTTP/DB, parametrizables por ambiente (5 pts).

## 7. Estándares de Performance

**Aplica si:** el diff toca código con loops, I/O, llamadas a servicios externos, cachés, o manejo de concurrencia/hilos.

**Criterios:**
- Sin operaciones bloqueantes dentro de flujos async/reactivos (3 pts).
- Recursos (conexiones, streams, clientes) se cierran correctamente (try-with-resources o equivalente) (3 pts).
- Cachés (si existen) tienen política de expiración/tamaño, no crecimiento ilimitado (2 pts).
- Sin queries/N+1 evidentes o lecturas repetidas innecesarias del mismo dato (2 pts).

## 8. Estándares de Rutas / Infraestructura

**Aplica si:** el diff define o modifica rutas expuestas (OpenShift Route, Ingress, API Gateway) u hostnames.

**Criterios:**
- Nomenclatura y estructura de la ruta sigue el estándar de la organización (5 pts).
- No se exponen rutas/hosts de ambientes no productivos hacia el exterior por error (5 pts, violación = CRÍTICO).

---

## Cómo personalizar este catálogo
- Agrega/quita categorías según lo que tu organización realmente audite.
- Ajusta los pesos de cada criterio para que sumen 10 por categoría.
- Si tu organización tiene un formato de nomenclatura de variables específico (ej. `[ACRONYM]_[TOOL]_[COMPONENT]_[ATTRIBUTE]`), documéntalo aquí explícitamente para que el agente lo verifique literalmente en vez de con criterio genérico.
