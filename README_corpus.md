# Corpus — MineSight DD

A diferencia de un test que entrega un Drive con PDFs estáticos, en este test el **corpus se construye automáticamente** desde la web de relación con inversionistas de **Mineros S.A.** (Colombia).

---

## Fuente única del corpus

**Página de informes financieros de Mineros**:
> https://www.mineros.com.co/es-co/inversionistas/informes-financieros

La página agrupa los informes por **año (2021-2026)** y por **trimestre (Q1, Q2, Q3, Q4)** + **Formulario de Información Anual** por año.

El candidato debe descargar programáticamente **todos los PDFs disponibles** desde esta URL. Ver detalles en `CASE_BRIEF.md` → **Bloque 1: Pipeline de adquisición automatizada**.

---

## Procesamiento

Aunque la descarga es de todos los años, el procesamiento (extracción + consolidación + dashboard) se ejecuta **únicamente sobre los 4 trimestres de 2025**. El código debe estar diseñado para procesar cualquier año cambiando un parámetro — esto evalúa criterio de ingeniería (no hardcodeo).

---

## Idioma

Los informes de Mineros están en **español**. El candidato debe configurar herramientas de extracción y, si usa LLMs, modelos que manejen español correctamente.

---

## Notas para el evaluador interno

### Antes de la primera evaluación

1. **Visitar manualmente la URL** y verificar que la página está viva, accesible, y que los enlaces a PDFs funcionan. Si la página fue rediseñada, ajustar `CASE_BRIEF.md` para reflejarlo.
2. **Descargar manualmente los 4 PDFs trimestrales de 2025** (Q1-Q4) y leerlos para tener una vista propia de qué KPIs son razonables esperar.
3. **Identificar 5 cifras de tabla específicas** (ej. producción de oz en Q1 2025, AISC en Q3 2025) que se usarán para verificar la precisión de la extracción del candidato. Ver `_internal/CALIBRATION_QUERIES.md`.
4. **Documentar la versión del sitio** (fecha de visita) en este archivo.

### Si la página tiene anti-bot / JS dinámico

- Si **tú no pudiste descargar fácilmente** los PDFs con un script simple (`requests` + `BeautifulSoup`), entonces **no descalifiques** a un candidato que documenta el mismo bloqueo y descarga manualmente.
- Lo que sí debe penalizarse: que el candidato no intente nada (ni código, ni documentación).

### Histórico de verificación del corpus

| Fecha visita | Quién verificó | Años visibles en la página | Notas / cambios en el sitio |
|---|---|---|---|
| ____ | ____ | 2021-2026 (al 2026-05-25) | rellenar antes del primer uso |

---

## ¿Por qué Mineros y no una compañía internacional?

- **Realismo regional**: alineado al mercado donde opera Sun Valley.
- **Idioma**: los analistas de Sun Valley trabajan en español; el sistema debe funcionar en español.
- **Tamaño accesible**: el corpus de Mineros es manejable en 3 días, a diferencia de mineras majors con miles de páginas.
- **Datos públicos reales**: cifras verificables, no inventadas.
- **Estructura clara**: la página agrupa informes por año y trimestre, ideal para evaluar un pipeline de descarga estructurado.
