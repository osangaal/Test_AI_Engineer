# Corpus — MineSight DD

A diferencia de un test que entrega un Drive con PDFs estáticos, en este test el **corpus se construye automáticamente** desde la web de relación con inversionistas de **Mineros S.A.** (Colombia).

---

## Fuente única del corpus

**Página de informes financieros de Mineros**:
> https://www.mineros.com.co/es-co/inversionistas/informes-financieros

El candidato debe descargar programáticamente desde esta URL. Ver detalles en `CASE_BRIEF.md` → **RF-0: Pipeline de adquisición**.

---

## ¿Qué tipos de documentos hay disponibles?

La página de Mineros publica habitualmente:

- **Informes Anuales** (PDF, ~100-200 págs/año)
- **Informes Trimestrales de Resultados** (PDF, ~20-40 págs)
- **Resúmenes / Earnings releases** (PDF cortos)
- Potencialmente reportes técnicos o de sostenibilidad

El candidato debe descargar **mínimo 5 documentos** — un mix de anual + trimestrales recientes.

---

## Idioma

Los informes de Mineros están en **español**. El candidato debe configurar embeddings y prompts en consecuencia (modelos multilingual u OpenAI/Anthropic que manejan español nativamente).

---

## Notas para el evaluador interno

### Antes de la primera evaluación

1. **Visitar manualmente la URL** y verificar que la página está viva, accesible, y que los enlaces a PDFs funcionan. Si la página fue rediseñada, ajustar `CASE_BRIEF.md` para reflejarlo.
2. **Descargar manualmente** los mismos PDFs que esperamos que descargue el candidato (~5-10 informes recientes). Esto es necesario para:
   - Resolver manualmente las queries de calibración (ver `CALIBRATION_QUERIES.md`)
   - Tener un baseline propio para comparar
3. **Documentar la versión del sitio** (fecha de visita) en este archivo. Si la página cambia mucho, las queries de calibración pueden quedar desactualizadas.

### Si la página tiene anti-bot / JS dinámico

- Si **tú no pudiste descargar fácilmente** los PDFs con un script simple (`requests` + `BeautifulSoup`), entonces **no descalifiques** a un candidato que documenta el mismo bloqueo y descarga manualmente.
- Lo que sí debe penalizarse: que el candidato no intente nada (ni código, ni documentación).

### Histórico de verificación del corpus

| Fecha visita | Quién verificó | # documentos descargados | Notas / cambios en el sitio |
|---|---|---|---|
| ____ | ____ | ____ | rellenar antes del primer uso |

---

## ¿Por qué Mineros y no una compañía internacional?

- **Realismo regional**: alineado al mercado donde opera Sun Valley.
- **Idioma**: los analistas de Sun Valley trabajan en español; el sistema debe funcionar en español.
- **Tamaño accesible**: el corpus de Mineros es manejable en 3 días, a diferencia de mineras majors con miles de páginas.
- **Datos públicos reales**: cifras verificables, no inventadas.
