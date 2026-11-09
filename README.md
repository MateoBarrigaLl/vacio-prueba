# Agente de Regresión Inteligente

Kit de **agentes de Copilot (VS Code)** para automatizar el análisis de regresión de un PR o rama: qué cambió, qué impacto tiene, qué tan riesgoso es, qué pruebas correr primero (con veredicto por caso y artefactos ejecutables) y si cumple los estándares de la organización.

## Estructura

```
.github/
├── agents/
│   ├── qa-regression-orchestrator.agent.md    # Orquestador principal (punto de entrada)
│   ├── pr-diff-analyzer.agent.md              # Etapa 1: extrae el diff estructurado
│   ├── impact-mapper.agent.md                 # Etapa 2: mapea el blast radius
│   ├── risk-assessor.agent.md                 # Etapa 3: matriz de riesgo
│   ├── test-prioritizer.agent.md              # Etapa 4: veredicto por test (MANTENER/ACTUALIZAR/CREAR/ELIMINAR) + score
│   ├── test-case-generator.agent.md           # Etapa 5: genera regression-suite.txt + generated-tests.py
│   ├── standards-compliance-validator.agent.md# Etapa 6: gate de cumplimiento de estándares
│   └── qa-report-writer.agent.md              # Etapa 7: reporte final en Markdown
└── standards/
    ├── pr-standards-catalog.md                # Catálogo de estándares (configurable por equipo)
    └── rag-topic-config.md                    # Topic de RAGnarōk para la regresión rápida de test-prioritizer (configurable)

demo-app/                                      # App de ejemplo (a propósito acoplada) para probar los agentes
```

## Cómo usarlo

1. Copia la carpeta `.github/` a la raíz de tu repositorio (o clona este repo dentro de él).
2. En VS Code, con el chat de Copilot en modo **Agent**, invoca al orquestador escribiendo `@QA Regression Orchestrator` (o seleccionándolo desde el selector de agentes) y dale el contexto del cambio a analizar, por ejemplo:

   ```
   analiza mi rama feature/pagos contra develop
   ```

   ```
   analiza el PR #123
   ```

3. El orquestador coordina automáticamente a los 7 agentes especialistas (sin que tengas que invocarlos tú mismo) y entrega:
   - Un resumen ejecutivo en el chat.
   - Un reporte Markdown completo con el detalle de cada etapa.
   - Dos artefactos de prueba: `regression-suite-<id>-<fecha>.txt` (node ids listos para correr) y `generated-tests-<id>-<fecha>.py` (código real de los tests nuevos/actualizados).
   - Una **Decisión Global de Bloqueo (SI/NO)** y los 3-5 ítems P0 más críticos para empezar.

4. Si solo necesitas una etapa puntual, puedes pedirla directamente, por ejemplo "solo dime el riesgo" o "solo dime si cumple estándares", y el orquestador invocará únicamente al especialista necesario.

## Configurar los estándares

El archivo [`.github/standards/pr-standards-catalog.md`](.github/standards/pr-standards-catalog.md) define las categorías, criterios, pesos y umbrales que usa `standards-compliance-validator` para puntuar cada PR (pruebas unitarias, arquitectura, seguridad OWASP, resiliencia, performance, rutas/infra). Ajusta este catálogo para que coincida con la política real de tu equipo u organización antes de usarlo en producción.

## Configurar la regresión rápida con RAGnarōk (opcional)

`test-prioritizer` puede consultar la herramienta `ragQuery` de RAGnarōk para sugerir una **regresión rápida** (mínimo dos consultas: símbolos modificados y consumidores/cadenas de `impact-mapper`), independiente de su score heurístico habitual. Para activarla, define el topic ya indexado en [`.github/standards/rag-topic-config.md`](.github/standards/rag-topic-config.md). Si el `topic` queda vacío, esta fase se omite automáticamente y el resto del agente sigue funcionando igual. El modo activo por defecto es `ragnarok_0_3_vector` (POC verificada con RAGnarōk 0.3.0: `retrievalStrategy: vector`, `similarity` mostrado como `legacy_vector_similarity`, nunca como probabilidad); el modo `cross_encoder_probability` se conserva intacto para una futura versión con reranker calibrado.

## Probar con el demo-app

`demo-app/` es una API Flask intencionalmente acoplada (estado global compartido, lógica de negocio mezclada con la capa HTTP) usada como caso de prueba. La rama `feature/coupons-and-cancel` introduce cambios riesgosos sobre `main` para validar que los agentes los detecten. Ver [demo-app/README.md](demo-app/README.md).

## Notas

- Los agentes de diff, impacto y riesgo son de **solo lectura**; no ejecutan comandos destructivos de git (`reset --hard`, `push --force`, merges).
- `test-prioritizer` sí ejecuta las pruebas existentes del repo (cuando es posible) para confirmar resultados reales, además de priorizarlas.
- El **Score de relevancia** de la tabla "Veredicto por test" (`0.00`–`1.00`) es una **heurística de trazabilidad de código** — mide qué tanto se relaciona cada test con el código cambiado en el PR, **no** es cobertura ni % de aciertos. `1.00` = el test invoca/asserta directamente el símbolo cambiado; el Score y el `PASS`/`FAIL` son columnas independientes.
- Si corres el análisis varias veces el mismo día, el reporte y sus artefactos se guardan con sufijo incremental (`-v2`, `-v3`, ...) para no sobrescribir los previos.
- La aprobación final de un PR sigue siendo responsabilidad del revisor humano — estos agentes son una ayuda de análisis, no un reemplazo del juicio del equipo.
