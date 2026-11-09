# Configuración de RAGnarōk para la Regresión Rápida

Este archivo define el `topic` que el agente `test-prioritizer` usa al invocar la herramienta `ragQuery` en su fase complementaria **"Regresión rápida asistida por RAGnarōk"** (ver [test-prioritizer.agent.md](../agents/test-prioritizer.agent.md)).

Es **configurable por equipo/proyecto**, igual que [pr-standards-catalog.md](./pr-standards-catalog.md). El agente NUNCA debe hardcodear un nombre de topic; siempre lo lee de aquí.

## Topic

```
topic: "qa-agent-regression-code"
```

> Reemplaza el valor de arriba con el nombre EXACTO del topic ya indexado en RAGnarōk para este repositorio (por ejemplo, el topic bajo el cual indexaste el código fuente y los tests de `demo-app/`). Si RAGnarōk hace fuzzy-match de topics semánticamente similares, aun así especifica aquí el nombre real usado al indexar — no un nombre inventado.

## Modo activo: `ragnarok_0_3_vector` (POC verificada con RAGnarōk 0.3.0)

```
mode: "ragnarok_0_3_vector"
```

Hallazgos reales verificados con RAGnarōk 0.3.0 (no asumidos): `ragQuery` devuelve `similarity` pero **no** `metadata.scoreKind`; `retrievalStrategy: vector` da scores útiles; `bm25` y `ensemble` devuelven `similarity == 0`; `hybrid` ordena peor para este escenario. Por eso, mientras el `mode` activo sea `ragnarok_0_3_vector`:

- `retrievalStrategy`: **`vector`** (obligatorio — `bm25`, `ensemble` e `hybrid` NO se usan en este modo).
- `topK`: `10`.
- `similarity` se trata como **`legacy_vector_similarity`**, explícitamente NO como una probabilidad calibrada.
- Umbrales **PROVISIONALES de POC** (revisar cuando exista un reranker calibrado): ALTA `>= 0.65`, MEDIA `>= 0.50` y `< 0.65`, BAJA `< 0.50`.
- Un `similarity` alto por sí solo **no** demuestra que el test alcance el símbolo modificado: la inclusión final SIEMPRE combina el nivel (ALTA/MEDIA/BAJA) con un camino de impacto verificado (evidencia de `impact-mapper`), según la matriz de decisión de `test-prioritizer.agent.md`.
- `test-prioritizer` debe invocar `ragQuery` **al menos dos veces** por corrida: (1) por los símbolos/comportamientos modificados, (2) por los consumidores/cadenas de impacto entregados por `impact-mapper`. Los resultados de ambas consultas se combinan en una sola lista y se deduplican por `<archivo>::<test>` conservando el `similarity` máximo antes de clasificar.

## Modo futuro (kept intact): `cross_encoder_probability`

```
mode: "cross_encoder_probability"
```

Política ORIGINAL, preservada intacta para cuando RAGnarōk incorpore un reranker final calibrado. Mientras ese `mode` esté activo:

- `retrievalStrategy`: `hybrid`
- `topK`: `20`
- Solo `metadata.scoreKind == "cross_encoder_probability"` habilita clasificación ALTA/MEDIA/BAJA con decisión automática (umbrales `0.80`/`0.55`). Cualquier otro `scoreKind` (`vector_similarity`, `weighted_fusion`, `bm25_normalized`) o su ausencia se trata como relevancia RELATIVA: se muestra el `similarity` pero requiere revisión manual, sin auto-incluir ni auto-omitir.

## Comportamiento si falta configuración

- Si este archivo no existe, o el campo `topic` está vacío (`""`): `test-prioritizer` **omite** la fase RAGnarōk, continúa con su flujo heurístico existente sin cambios, y advierte explícitamente en su output que el topic no está configurado.
- Si `ragQuery` responde que el topic no tiene documentos indexados: se informa que aún no hay documentos indexados para esa fase, sin bloquear el resto del análisis.
- Si `ragQuery` falla o no está disponible: se advierte el fallo y se continúa con el flujo heurístico existente (degradación controlada, nunca se detiene el análisis completo por esto).
- Si `ragQuery` devuelve `topicMatched` distinto de `"exact"` (p. ej. `"similar"` o `"fallback"`), o `topicName` no coincide EXACTAMENTE con el `topic` de este archivo: los resultados **no se procesan** (RAGnarok pudo haber usado un topic distinto por similitud); se informa la incompatibilidad y el resto del agente sigue funcionando.
- La clasificación ALTA/MEDIA/BAJA depende del `mode` activo (ver secciones arriba): con `ragnarok_0_3_vector` se basa en `legacy_vector_similarity` (umbrales `0.65`/`0.50`, provisionales de POC); con `cross_encoder_probability` solo aplica cuando `metadata.scoreKind == "cross_encoder_probability"` (umbrales `0.80`/`0.55`) — cualquier otro `scoreKind` o su ausencia se trata como relevancia RELATIVA, sin auto-incluir ni auto-omitir.