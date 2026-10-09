# Revisión de `PROMPT_CLAUDE_CODE.md`

Revisión del prompt antes de lanzarlo, contrastando sus reglas contra los datos reales de `datos/clientes_ledmon.csv`. Nada de esto es una ejecución de la auditoría: es una revisión de viabilidad y coherencia del propio prompt.

## 1. Contradicción real en la Fase 0 (bloqueante)

El encabezado dice:
> **Fase 0: preparación (enséñame el resultado antes de seguir)**

Y dos líneas después:
> Después **continúa sin esperar mi confirmación**.

Son dos instrucciones opuestas. Quien ejecute el prompt tiene que decidir por su cuenta cuál prevalece (probablemente la segunda, porque es más explícita y aparece después), pero tal como está escrito, un ejecutor literal podría quedarse parado esperando una confirmación que la Fase 0 dice que no hace falta. **Hay que elegir una de las dos y borrar la otra.** Si la intención es "avisa si algo es grave, pero no esperes a que conteste", conviene decirlo así explícitamente (p. ej. "para solo si más del 20% de las webs no están OK").

## 2. `cliente_activo` está vacío en el 100% de las 202 filas

Comprobado sobre el CSV real: las 202 filas tienen `cliente_activo` en blanco (el README ya lo avisa: "Vacío. Rellenar con `no`..."). Es decir, **la regla 4 ("salta las filas con `cliente_activo = no`") no excluye a nadie todavía**, porque nadie ha rellenado esa columna. El prompt no dice qué hacer en ese caso:

- ¿Debe Claude Code rellenar él mismo `cliente_activo=no` en la Fase 0 a partir de lo que detecte (dominio aparcado, redirige a otro negocio, etc.), o eso lo decide un humano de Ledmon antes?
- Si nadie la rellena, ¿se audita a las 201 filas de tipo `Cliente` igualmente, incluidas las que ya no son clientes reales?

Conviene decidirlo explícitamente en el prompt (sugerencia: en la Fase 0, proponer una lista de candidatas a `cliente_activo=no` a partir de la clasificación de estado, y seguir con todas mientras tanto, marcándolas pero no excluyéndolas de raíz).

## 3. Tamaño real de la auditoría (para dimensionar expectativas)

Con los 201 dominios de tipo `Cliente` (el único no-cliente es 1 fila "Ledmon (agencia)", no varias como sugiere el README "web propia de Ledmon **o de Víctor**" — en los datos actuales no hay ninguna fila de Víctor):

| nivel | nº webs | páginas/web | total páginas (Fase 1.3) |
|---|---|---|---|
| A | 48 | 30 | 1.440 |
| B | 59 | 15 | 885 |
| C | 82 | 6 | 492 |
| D | 12 | 3 (home+robots+sitemap) | 36 |
| **Total** | **201** | | **~2.850 páginas** |

A eso hay que sumar, por dominio (no por página): robots.txt + sitemap + muestra de 20 URLs del sitemap (Fase 1.2), 3 peticiones con UA de bots de IA (Fase 2.2) → **~600 peticiones más**, y para los 107 dominios A+B, 1 llamada a PageSpeed Insights cada uno (Fase 1.9) → **107 llamadas a PSI**. Para los 48 de nivel A, 2–3 búsquedas con WebSearch (Fase 2.7) → **~120 búsquedas**.

Esto no es inviable, pero sí es una auditoría de varios miles de peticiones HTTP más generación de 107 fichas individuales (`clientes/<dominio>.md`) y un Excel con, probablemente, varios miles de filas de hallazgos. **El prompt no dice que esto va a necesitar varias sesiones/reanudaciones de Claude Code**, aunque sí prevé caché y commits cada 20 webs (buena señal de que ya se intuía). Sugerencia: decirlo explícitamente ("esta auditoría no se termina en una sola sesión; apóyate en `cache/` y en los commits para reanudar") para que quien la lance no se sorprenda si se corta a mitad.

## 4. La prueba de bloqueo real a bots de IA (Fase 2.2) choca con la Regla 2

La regla 2 dice "Respeta robots.txt para rastrear", pero la Fase 2.2 pide pedir la home **con el User-Agent exacto de GPTBot/ClaudeBot/PerplexityBot**, que es justo la comprobación que tiene sentido hacer **incluso si robots.txt les bloquea** (de hecho ese es el caso interesante: ver si el bloqueo del robots.txt se refleja también a nivel de servidor/WAF, o si solo es de boquilla). Tal como está redactado, no queda claro si esa única petición de diagnóstico por bot y por dominio es una excepción admitida a "respeta robots.txt", o si hay que saltarla cuando el robots.txt ya los bloquea (lo cual anularía el sentido del punto 2 de la Fase 2). **Conviene decirlo de forma explícita**: "esta petición de diagnóstico se hace siempre, bloquee o no el robots.txt, porque es precisamente lo que se quiere comprobar; no cuenta como rastreo sistemático".

También vale la pena una advertencia ética/legal de una línea: identificarse con el user-agent exacto de GPTBot/ClaudeBot/PerplexityBot en 201 dominios ajenos (aunque sean clientes propios) es indistinguible, visto desde el log del servidor, de ser el bot real. Para los propios clientes de Ledmon no debería haber problema, pero conviene dejar constancia en el prompt de que se hace con conocimiento de causa y solo contra la propia cartera.

## 5. Umbral de fecha fijo que rompe la repetibilidad trimestral

Fase 3 usa `sc_ultimo_dato anterior al 27/09/2026` como corte fijo. Esa fecha tiene sentido ahora mismo (está calculada a partir de la fecha de extracción del CSV, el 07/10/2026, con la ventana de 90 días), pero el prompt pide explícitamente que los scripts de `scripts/` sirvan **"para repetir la auditoría cada trimestre con el mismo CSV"** (y presumiblemente con un CSV nuevo cada trimestre). Un corte en fecha absoluta no se puede reutilizar sin editarlo a mano cada vez. Sugerencia: expresarlo en relativo ("más de N días antes de la fecha de extracción del CSV" o "más de N días antes de hoy"), no como fecha fija.

## 6. La rúbrica de puntuación no tiene ni un esqueleto de partida

El prompt pide "una rúbrica explícita: bloques, pesos y criterios... reproducible para repetirla cada trimestre", pero no propone ni los bloques ni un reparto de pesos orientativo. Quien ejecute el prompt se inventa la rúbrica entera desde cero, lo que hace que la nota de 0 a 100 dependa completamente de un criterio no fijado de antemano en el prompt. Para que de verdad sea comparable trimestre a trimestre, convendría fijar aquí (aunque sea de forma provisional, ajustable después) los bloques que entran en la nota SEO y en la nota GEO y su peso relativo — por ejemplo algo como: Técnico 20%, Indexabilidad 15%, On-page 20%, Schema 10%, Local 15%, Contenido/E-E-A-T 20% para SEO; y Accesibilidad a bots de IA 25%, Entidad clara 20%, Contenido citable 35%, Visibilidad observada 20% para GEO. Así la primera ejecución fija el estándar y las siguientes lo heredan en vez de reinventarlo.

## 7. Gotcha técnico de los CSV (para quien escriba el script, no es un error del prompt)

Los 4 CSV usan `;` como separador y UTF-8 **con BOM** (confirmado: la primera cabecera de cada archivo llega como `﻿dominio`, no `dominio`, si se lee con un `csv.reader` normal). Si el script de la Fase 0 no abre los ficheros con `encoding='utf-8-sig'`, la columna `dominio` no se reconocerá por su nombre y el cruce con los otros CSV fallará en silencio. Vale la pena añadir esta nota al prompt o al README para ahorrar ese tropiezo.

## 8. Qué hacer con la caché y los commits cada 20 webs

La regla 5 pide cachear todo en `cache/` y hacer commit del avance cada 20 webs, pero no dice si `cache/` debe entrar en el control de versiones o quedar fuera (`.gitignore`). Si se cachea el HTML completo y las cabeceras de imágenes de ~2.850 páginas, el repositorio puede crecer varios cientos de MB o más en pocos commits. Sugerencia: aclarar que `cache/` se ignora en git (solo vive en disco, para poder reanudar dentro de la misma sesión/entorno) y que lo que se commitea cada 20 webs es el **progreso en `informe/`** (fichas, filas del Excel/CSV intermedio), no la caché HTTP en sí.

## 9. Pequeños puntos de redacción, sin impacto en la ejecución

- El README dice que `tipo` puede ser "web propia de Ledmon **o de Víctor**", pero en los datos actuales solo hay una fila que no es `Cliente` ("Ledmon (agencia)"). No es un error, solo una desalineación entre la documentación genérica y el CSV actual — por si se esperaba más de una fila ahí.
- Fase 1.3 menciona comprobar que los hreflang "estén bien enlazados entre sí" sin precisar el criterio (reciprocidad de las etiquetas, uso de `x-default`, etc.). No es grave, pero como el resto del prompt es muy explícito en criterios, aquí se nota la falta de uno.
- La Fase 1.9 (PageSpeed) dice "sin clave, ve despacio para no quedarte sin cuota" pero no dice qué hacer si de verdad se agota la cuota a mitad de los 107 dominios A+B (¿se para esa sección y se sigue con el resto de fases, marcando como `no verificado`? Probablemente sí, por la regla 6, pero conviene decirlo).

## Resumen (para decidir antes de lanzar)

1. **Hay que resolver la contradicción de la Fase 0** (¿se espera confirmación o no?) antes de ejecutar nada; es lo único realmente bloqueante.
2. Decidir qué hacer con `cliente_activo` vacío en las 202 filas.
3. Avisar de que esto son varias sesiones de trabajo, no una.
4. Aclarar que la prueba de bloqueo a bots de IA se hace aunque el robots.txt los bloquee.
5. Cambiar el corte de fecha fijo de la Fase 3 por uno relativo.
6. Dar un esqueleto inicial de pesos para la rúbrica de puntuación.
7. Nota técnica: CSV con BOM, usar `utf-8-sig` al leer.
8. Decidir si `cache/` entra en git o se ignora.

Con esos ocho puntos resueltos, el prompt es ejecutable tal cual: las reglas de rastreo, los límites por nivel y los entregables están bien definidos y son consistentes con los datos reales del CSV.
