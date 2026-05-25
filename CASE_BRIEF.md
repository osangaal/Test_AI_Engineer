# Test Técnico — Ingeniero de IA

**Sun Valley Investment — Posición: Ingeniero de IA, Especialista en desarrollo e implementación**

Bienvenido. Este documento describe el caso técnico que debes resolver. Léelo completo antes de empezar.

---

## 1. Contexto de negocio

Sun Valley Investment es un fondo que invierte en activos mineros. Cada oportunidad requiere revisar decenas de informes trimestrales y anuales para construir una vista consolidada de los KPIs financieros y operacionales de la compañía. Hoy un analista hace esto manualmente, copiando cifras de PDFs a Excel — proceso lento y propenso a errores.

Tu misión: construir **MineSight DD**, un sistema end-to-end que automatice la extracción, consolidación y visualización de indicadores clave a partir de los informes públicos de una compañía minera.

La compañía objetivo es **Mineros S.A.** (Colombia), por su disponibilidad pública de informes y relevancia regional. Sus informes están publicados en:

> https://www.mineros.com.co/es-co/inversionistas/informes-financieros

La página agrupa informes por año (2021-2026) y por trimestre (Q1-Q4) + informes anuales.

---

## 2. El sistema a construir (3 bloques)

### Bloque 1 — Pipeline de adquisición automatizada

Script que **descarga todos los PDFs disponibles** en la página de Mineros, **sin filtro de año** (todos los años visibles). Debe ser:
- **Idempotente**: no re-descargar lo que ya está localmente
- **Robusto**: retries con backoff, User-Agent identificable, respeto a `robots.txt`
- **Bien organizado**: estructura clara en disco (ej. `data/raw/2025/Q1/...`)
- **Trazable**: logs de qué descargó, desde qué URL, qué tamaño

**Si encuentras bloqueos del sitio** (anti-bot, JS dinámico, captcha) que no puedes resolver razonablemente: documenta el bloqueo en tu README, muestra el código que probaste, y descarga manualmente. **No serás penalizado** si el bloqueo es real y está bien documentado.

### Bloque 2 — Extracción + consolidación de KPIs

Sobre los PDFs descargados, debes:

1. **Detectar y extraer todas las tablas** presentes en los informes. Aquí esperamos que uses IA donde aporte valor (LLMs para parsear tablas complejas, vision models para tablas en imágenes, etc.).
2. **Proponer un conjunto de KPIs financieros + operacionales** que consideres pertinentes para evaluar a Mineros como inversión. Tienes **libertad total** para decidir cuáles — esperamos que justifiques tus elecciones.
3. **Consolidar esos KPIs en una sola tabla normalizada** (CSV, Parquet, SQLite — tú decides).

**Alcance del procesamiento**: ejecuta el bloque 2 **únicamente sobre los 4 trimestres de 2025** (Q1, Q2, Q3, Q4). Esto es para mantener el trabajo manejable.

**IMPORTANTE — reproducibilidad**: tu código **NO** debe estar hardcodeado a 2025. Debe estar diseñado para procesar **cualquier año** (o varios años) cambiando un parámetro. En tu README explica cómo correrlo para, por ejemplo, los 4 Qs de 2024.

> Esto evalúa criterio de ingeniería: no queremos un script de un solo uso, queremos un pipeline reusable.

### Bloque 3 — Visualización para audiencia gerencial

Construye una interfaz visual sobre la tabla consolidada. **Formato libre**: Streamlit, Gradio, dashboard web, notebook con plots, app FastAPI con frontend mínimo, lo que prefieras.

La audiencia es **gerencial / toma de decisiones de inversión** — no técnica. Las visualizaciones deben:
- Permitir ver la evolución trimestral de los KPIs
- Permitir comparativos (ej. Q3 2025 vs Q3 2024 si tienes ambos)
- Resaltar tendencias o alertas relevantes para un analista de inversión

**Justifica tu elección de formato** en el reporte (por qué Streamlit y no Gradio, por qué incluir tal gráfica, etc.).

---

## 3. Requerimientos no-funcionales

- **Logging**: estructurado, con trazabilidad de cada PDF descargado y cada tabla extraída
- **Manejo de errores** específico (no `except Exception` genéricos); retries con backoff en descargas HTTP
- **Modularidad**: separación razonable (`acquisition/`, `extraction/`, `consolidation/`, `dashboard/`)
- **Tests**: mínimo **3 tests unitarios significativos**
- **Tipado**: type hints en funciones públicas
- **Reproducibilidad**: dependencias pinneadas (`requirements.txt` o `pyproject.toml`); seeds fijos donde aplique
- **Configuración por parámetros**: el año a procesar debe ser un parámetro, no estar hardcodeado

---

## 4. Stretch goals opcionales (bonus +15%)

Diferencian a candidatos top. **No son obligatorios** dado el plazo corto.

- **S-1** — **Agente conversacional sobre los datos**: agente (LangChain/LangGraph) con tool de SQL/lookup sobre la tabla consolidada, que responde preguntas en lenguaje natural ("¿en qué trimestre fue mayor el AISC?"). Incluye citas al PDF/tabla original.
- **S-2** — **BD relacional normalizada**: persistir la tabla en PostgreSQL/SQLite con esquema definido + script de migración.
- **S-3** — **Procesamiento multi-año real**: además de 2025, procesa al menos otro año completo (2024 o 2023) y muestra comparativo anual en el dashboard.
- **S-4** — **Despliegue real** en Azure / Render / Railway / Streamlit Cloud con URL pública.
- **S-5** — **Evaluación de calidad de extracción**: golden set de 10+ cifras verificadas manualmente vs lo que extrajo tu sistema, con reporte de precisión.

---

## 5. Entregables

### 5.1 Repositorio Git privado

Invita como colaborador a `{{recruiting-user}}`. Estructura sugerida:

```
minesight-dd/
├── README.md                    # Cómo correr en <5 min, supuestos, horas reales
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── .env.example
├── src/
│   ├── acquisition/             # descargador de informes
│   ├── extraction/              # parsing + detección de tablas
│   ├── consolidation/           # normalización y construcción de tabla de KPIs
│   ├── dashboard/               # visualización
│   └── config/
├── tests/
├── data/
│   ├── raw/                     # PDFs descargados (gitignored)
│   └── processed/               # tabla consolidada
└── docs/
    ├── REPORT.md                # ver 5.2
    └── slides.pdf               # ver 5.3
```

Tu README debe incluir: instrucciones de ejecución, **horas reales invertidas**, supuestos hechos, **limitaciones encontradas al scrapear** (si aplica) y **cómo correr el pipeline para otro año**.

### 5.2 Reporte (`docs/REPORT.md`)

Secciones obligatorias (2-4 páginas):

1. **Resumen ejecutivo** (1 párrafo): ¿está listo para producción? Sí/No y por qué.
2. **Pipeline de adquisición**: cuántos PDFs descargó, problemas encontrados, cómo los resolvió.
3. **KPIs propuestos**: lista de los indicadores que consolidaste, **con justificación de cada uno desde la perspectiva de un analista de inversión minera**.
4. **Calidad de la extracción**: cómo verificaste que las cifras extraídas son correctas. Si hiciste evaluación cuantitativa (golden set), incluir resultados.
5. **Decisiones técnicas + trade-offs**: stack de extracción, elección de modelo LLM si aplica, normalización entre PDFs (los nombres de columnas pueden variar entre Qs).
6. **Limitaciones conocidas**: qué NO funciona aún.
7. **Roadmap**: qué priorizarías con 1 semana adicional.

### 5.3 Presentación (slides PDF)

Entre **6 y 10 slides**. Estructura mínima:

1. Problema en **lenguaje de inversión**, no técnico
2. Arquitectura de alto nivel (diagrama: descarga → extracción → consolidación → dashboard)
3. KPIs propuestos + justificación (1 slide)
4. Demo del dashboard (screenshots de las visualizaciones más importantes)
5. Decisiones técnicas + trade-offs (1 slide)
6. Limitaciones + Roadmap
7. **Costo estimado de operación** si el sistema corriera mensualmente

Video Loom de 3-5 min explicando las slides es **opcional pero valorado**.

---

## 6. Rúbrica de evaluación (100 pts + 15 bonus)

Te compartimos los pesos para que orientes tu esfuerzo:

| Competencia | Peso |
|---|---|
| A. Pipeline de adquisición (descarga robusta + organización) | **15 pts** |
| B. Extracción de tablas (precisión + uso inteligente de IA) | **25 pts** |
| C. KPIs propuestos + criterio de negocio + normalización | **25 pts** |
| D. Dashboard / visualización (utilidad para audiencia gerencial) | **15 pts** |
| E. MLOps (Docker, empaquetado, reproducibilidad multi-año) | **10 pts** |
| F. Calidad de código (modularidad, tests, logs) | **5 pts** |
| G. Comunicación (reporte + slides) | **5 pts** |
| Stretch goals (S-1 a S-5) | **+15 pts** |

---

## 7. Criterios de descalificación inmediata

Tu test será rechazado sin revisión profunda si ocurre cualquiera de:

1. El código no ejecuta siguiendo tu README en una máquina limpia
2. **Las cifras extraídas en la tabla consolidada no coinciden con los PDFs originales** (panel verifica una muestra de 5 cifras)
3. **El código está hardcodeado a 2025** y no puede procesar otro año
4. Secrets (API keys) commiteados en git history
5. Plagio evidente
6. Falta uno de los tres entregables core (repo, reporte, slides)
7. No intento alguno de pipeline de adquisición automatizada (ni código, ni documentación de por qué fue manual)
8. Dashboard ausente o solo capturas estáticas del código

---

## 8. Reglas

- **IA-assisted coding permitido** (Copilot, Cursor, ChatGPT). **Pero** debes poder defender cada decisión técnica en la sesión sincrónica.
- **Documenta tus supuestos**. Si algo es ambiguo, decide tú y justifica. Esto incluye **qué KPIs elegiste y por qué**.
- **Honestidad sobre tiempo**: reporta horas reales en el README.
- **Stack LLM libre**: OpenAI, Azure OpenAI, Anthropic, modelos open-source. El costo corre por tu cuenta.
- **Scraping responsable**: respeta `robots.txt`, agrega User-Agent identificable, delays razonables entre requests.

---

## 9. Plazo y entrega

- **Tiempo de trabajo estimado**: 8-12 horas efectivas
- **Plazo**: **3 días calendario** desde la recepción de este documento
- **Forma de entrega**: por email al recruiter, indicando link al repo + slides PDF adjuntos (o link público)

---

## 10. Después de la entrega

Programaremos una **sesión sincrónica de 30 minutos**:
- **10 min**: presentas el sistema (dashboard funcionando + decisiones clave)
- **20 min**: Q&A técnico sobre tus elecciones de KPIs, código y trade-offs

Esta sesión es el filtro principal contra abuso de LLM y para verificar profundidad real.

---

## 11. Contacto

Dudas **operativas** (problemas con el link de Mineros, formato de entrega): **ogaspar@oceloteminerals.com**

Dudas **técnicas**: no se responden — son parte de la evaluación.

**Éxitos.**

— Oscar Gaspar Alvarez
Lead AI · Sun Valley Investment
