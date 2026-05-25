# Test Técnico — Ingeniero de IA

**Sun Valley Investment — Posición: Ingeniero de IA, Especialista en desarrollo e implementación**

Bienvenido. Este documento describe el caso técnico que debes resolver. Léelo completo antes de empezar.

---

## 1. Contexto de negocio

Sun Valley Investment es un fondo que invierte en activos mineros junior y mid-cap. Cada oportunidad requiere revisar cientos de páginas de informes financieros, reportes técnicos y resultados trimestrales. Un analista junior gasta **8-12 horas por activo** extrayendo cifras clave: producción de onzas, AISC (All-In Sustaining Cost), reservas y recursos, EBITDA, costos operativos por mina, capex.

Tu misión: construir **MineSight DD**, un asistente de IA end-to-end que:

1. **Adquiera automáticamente** los informes publicados por una compañía minera desde su web de relación con inversionistas.
2. **Extraiga y estructure** las tablas y cifras clave de cada informe.
3. **Responda preguntas en lenguaje natural** con citas verificables (documento + página).

La compañía objetivo para este test es **Mineros S.A.** (Colombia), por su disponibilidad pública de informes y relevancia regional.

Ejemplos de preguntas que el asistente debe responder:
- "¿Cuántas onzas de oro produjo Mineros en 2023?"
- "Compara el AISC reportado en el último trimestre vs el mismo trimestre del año anterior."
- "¿Cuál es la guía de producción más reciente publicada por la compañía?"

Cada respuesta debe incluir **respuesta + cita (documento, página)**. Si la información no está en los informes, el asistente debe decirlo explícitamente. **Nunca inventar cifras.**

---

## 2. Alcance del test

### Tiempo
- **Trabajo efectivo estimado:** 6-10 horas
- **Plazo de entrega:** **3 días calendario** desde la recepción de este documento

### Stack
Eres libre de elegir librerías, frameworks y proveedor de LLM (OpenAI, Azure OpenAI, Anthropic, o modelos open-source vía Ollama / HuggingFace). El costo de API corre por tu cuenta — si quieres minimizarlo, puedes usar modelos open-source locales o tiers gratuitos.

### Fuente del corpus
**URL oficial de informes financieros de Mineros**:
> https://www.mineros.com.co/es-co/inversionistas/informes-financieros

Tu sistema debe **descargar los PDFs disponibles desde esa página automáticamente** (ver RF-0). Trabajarás sobre los informes que ahí publican: informes anuales, resultados trimestrales y cualquier reporte técnico que la compañía haya publicado.

**Nota importante**: si al implementar la descarga encuentras que la web tiene bloqueos (anti-bot, JS dinámico, captcha) que no puedes resolver razonablemente, **documenta el bloqueo en tu README** y descarga manualmente los informes. **No serás penalizado** por esto siempre y cuando: (a) lo documentes claramente, (b) muestres el intento técnico (código que probaste), y (c) tu pipeline siga siendo reproducible (ej. el script lee los PDFs desde una carpeta local).

---

## 3. Requerimientos funcionales (obligatorios)

| ID | Requerimiento |
|---|---|
| **RF-0** | **Pipeline de adquisición**: script que descarga automáticamente los PDFs publicados en la página de informes financieros de Mineros. Mínimo **5 documentos** (mix de anual + trimestrales). Debe ser idempotente (no re-descargar lo que ya está). Logs claros de qué descargó y desde qué URL. |
| **RF-1** | **Pipeline de ingesta + extracción de tablas**: parsing de PDFs con **manejo explícito de tablas** (donde viven las cifras de producción, AISC, costos). Chunking justificado. Indexación en base vectorial con metadata: `source`, `page`, `doc_type` (anual/trimestral), `period` (FY/Q). |
| **RF-2** | **RAG con mitigación de alucinaciones**: retrieval (vectorial o híbrido — justifica tu elección), prompt que fuerce citas obligatorias, manejo explícito de "no encontrado". |
| **RF-3** | **Agente con tools (LangChain o LangGraph)**, mínimo **2 herramientas**: `search_documents(query, filters)` y `calculate_financial_metric(metric, inputs)` (ej. AISC margin, comparación trimestral, sensibilidad simple a precio). |
| **RF-4** | **Evaluación con RAGAS**: golden dataset propio con **mínimo 10 preguntas** con respuestas y citas verificadas manualmente sobre los informes de Mineros. Reportar al menos 2 métricas RAGAS (`faithfulness` + `answer_relevancy`). **3 preguntas adversariales** cuyas respuestas no están en el corpus (medir abstención correcta). |
| **RF-5** | **Empaquetado**: `Dockerfile` + `docker-compose.yml` que levante todo con un solo comando, **incluida la ejecución del pipeline de adquisición + ingesta**. Interfaz: API FastAPI con `/query` y `/health`, **o** UI Streamlit/Gradio. `.env.example` provisto. Secrets nunca commiteados. |

**Sobre el idioma**: los informes de Mineros están en español. Tu sistema debe manejar correctamente embeddings y prompts en español (sugerencia: usar modelos multilingual o LLMs que manejen español nativamente).

---

## 4. Requerimientos no-funcionales

- **Logging**: estructurado, incluir tokens y latencia por llamada al LLM, y URL + filesize por cada PDF descargado
- **Manejo de errores** específico (no `except Exception` genéricos); retries con backoff en descargas HTTP
- **Modularidad**: separación razonable (`acquisition/`, `ingestion/`, `retrieval/`, `agent/`, `evaluation/`, `api/`)
- **Tests**: mínimo **3 tests unitarios significativos**
- **Tipado**: type hints en funciones públicas
- **Reproducibilidad**: dependencias pinneadas (`requirements.txt` o `pyproject.toml`)

---

## 5. Stretch goals opcionales (bonus +10%)

Diferencian a candidatos top. **No son obligatorios** dado el plazo corto.

- **S-1** — **BD normalizada de tablas extraídas**: persistir las tablas extraídas (producción, costos, etc.) en una BD estructurada (SQLite/Postgres) con esquema definido. Permite queries SQL además del RAG.
- **S-2** — **Retrieval híbrido** (vectorial + BM25) con re-ranking
- **S-3** — **Despliegue real** en Azure / Render / Railway con URL pública viva
- **S-4** — **Guardrails de validación numérica**: verifica que la cifra citada existe en la tabla/página antes de responder
- **S-5** — **CI/CD básico**: GitHub Actions con lint + tests

---

## 6. Entregables

### 6.1 Repositorio Git privado

Invita como colaborador a `{{recruiting-user}}`. Estructura sugerida:

```
minesight-dd/
├── README.md                    # Cómo correr en <5 min
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml               # o requirements.txt
├── .env.example
├── src/
│   ├── acquisition/             # scraper / descargador de Mineros
│   ├── ingestion/               # parsing + extracción de tablas
│   ├── retrieval/
│   ├── agent/
│   ├── api/
│   └── evaluation/
├── tests/
├── data/
│   ├── raw/                     # PDFs descargados (gitignored)
│   └── golden_set.jsonl         # 10+ Q&A curadas
└── docs/
    ├── EVAL_REPORT.md
    └── slides.pdf
```

Tu README debe incluir: instrucciones de ejecución, **horas reales invertidas**, supuestos relevantes y **cualquier limitación encontrada al scrapear la página de Mineros** (si aplica).

### 6.2 Reporte de evaluación (`docs/EVAL_REPORT.md`)

Secciones obligatorias (2-3 páginas):

1. **Resumen ejecutivo** (1 párrafo): ¿está listo para producción? Sí/No y por qué.
2. **Adquisición**: cuántos PDFs descargó, cómo (scraping o manual), problemas encontrados.
3. **Golden dataset**: cómo lo construiste, tipos de pregunta.
4. **Resultados RAGAS**: tabla con métricas reportadas.
5. **Análisis de errores**: top 3 fallos con root cause.
6. **Trade-offs**: decisiones clave (chunking, modelo, manejo de tablas en español) y por qué.
7. **Limitaciones conocidas**: qué NO funciona aún.
8. **Roadmap**: qué priorizarías con 1 semana adicional.

### 6.3 Presentación (slides PDF)

Entre **6 y 10 slides**. Estructura mínima:

1. Problema en **lenguaje de inversión**, no técnico
2. Arquitectura de alto nivel (diagrama: adquisición → ingesta → RAG → agente)
3. Decisiones técnicas + trade-offs (1-2 slides)
4. Demo (screenshots de 2 queries con citas reales sobre Mineros)
5. Resultados de evaluación
6. Limitaciones + Roadmap
7. **Costo estimado de operación** (USD/query)

Video Loom de 3-5 min **opcional pero valorado**.

---

## 7. Rúbrica de evaluación (100 pts + 10 bonus)

Te compartimos los pesos para que orientes tu esfuerzo:

| Competencia | Peso |
|---|---|
| A. RAG + mitigación de alucinaciones | **22 pts** |
| B. Agente + Python / LangChain o LangGraph | **18 pts** |
| C. Evaluación (RAGAS + análisis) | **18 pts** |
| D. Pipeline de adquisición + ingesta de tablas | **15 pts** |
| E. MLOps (Docker, empaquetado) | **12 pts** |
| F. Calidad de código (modularidad, tests, logs) | **8 pts** |
| G. Comunicación + visión de negocio | **7 pts** |
| Stretch goals (S-1 a S-5) | **+10 pts** |

---

## 8. Criterios de descalificación inmediata

Tu test será rechazado sin revisión profunda si ocurre cualquiera de:

1. El código no ejecuta siguiendo tu README en una máquina limpia
2. No hay evaluación cuantitativa (solo "funciona bien")
3. El sistema alucina cifras críticas verificables
4. No hay citas en las respuestas
5. Secrets (API keys) commiteados en git history
6. Plagio evidente
7. Falta uno de los tres entregables core (repo, reporte, slides)
8. No intento alguno de pipeline de adquisición (ni código, ni documentación de por qué fue manual)

---

## 9. Reglas

- **IA-assisted coding permitido** (Copilot, Cursor, ChatGPT). **Pero** debes poder defender cada decisión técnica en la sesión sincrónica.
- **Documenta tus supuestos**. Si algo es ambiguo, decide tú y justifica.
- **Honestidad sobre tiempo**: reporta horas reales en el README.
- **Sobre el scraping**: respeta `robots.txt` y agrega `User-Agent` identificable. No ataques el sitio (delays razonables entre requests). Si el sitio bloquea, documenta y sigue adelante con descarga manual.

---

## 10. Después de la entrega

Programaremos una **sesión sincrónica de 30 minutos**:
- **10 min**: presentas tu solución
- **20 min**: Q&A técnico sobre decisiones y código

Esta sesión es el filtro principal contra abuso de LLM. Prepárate para defender cada decisión.

---

## 11. Contacto

Dudas **operativas** (formato de entrega, problemas con el link de Mineros): `{{contacto-operativo}}`

Dudas **técnicas**: no se responden — son parte de la evaluación.

**Éxitos.**

— Equipo Sun Valley Investment
