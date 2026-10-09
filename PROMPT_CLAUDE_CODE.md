# Mega auditoría SEO + GEO de la cartera de clientes de Ledmon

## Contexto
Ledmon es una agencia de marketing digital de Galicia: web (sobre todo WordPress + Elementor), SEO, SEM y redes. Casi todos sus clientes son pymes locales de Pontevedra, Vigo, Santiago y O Salnés: hostelería, campings, clínicas, reformas, industria, comercio y despachos profesionales.

En este repositorio tienes:
- `README.md`: qué es cada fichero y cada columna. **Léelo primero.**
- `datos/clientes_ledmon.csv`: 202 webs, una por fila, con su tráfico (Analytics y Search Console, 90 días), sus contenedores de Tag Manager, su nivel de tráfico (A/B/C/D) y notas.
- `datos/analytics_propiedades.csv`, `datos/tagmanager_contenedores.csv` y `datos/search_console_propiedades.csv`: el detalle por herramienta.

Ledmon tiene plugins propios de WordPress que conviene detectar y aprovechar en las recomendaciones: **Ledmon Speed Caché** (`/wp-content/plugins/ledmon-speed-cache/`), **Ledmon FAQ Schema Guard** (`ledmon-faq-schema-guard`) y **Ledmon Traductor** (`ledmon-traductor`).

## Objetivo
Auditar cada web en dos frentes:
- **SEO:** técnico, on-page, local y contenido.
- **GEO (Generative Engine Optimization):** lo preparada que está una web para que la citen los buscadores y asistentes con IA, como Google AI Overviews / AI Mode, ChatGPT Search, Perplexity, Gemini o Claude.

Después, decirme **qué puede hacer Ledmon**:
- qué arreglar en cada cliente;
- qué desplegar en bloque en toda la cartera;
- qué servicios vender y a quién.

## Reglas
1. **Solo lectura.** No envíes formularios, no inicies sesión y no cambies nada en ninguna web.
2. **Rastreo educado:**
   - User-Agent `LedmonAuditBot/1.0 (+https://ledmon.com)`;
   - como máximo 1 petición por segundo y dominio, con varios dominios en paralelo (máx. 8);
   - timeout de 20 s y un reintento.
   - Respeta robots.txt para rastrear, pero descarga siempre el robots.txt.
3. **Límite de páginas por web** (sacadas del sitemap o, si no hay, enlaces internos de la home):

   | Nivel | Páginas |
   |---|---|
   | A | 30 |
   | B | 15 |
   | C | 6 |
   | D | home + robots + sitemap |
4. **Salta** las filas con `cliente_activo = no`. Las de tipo distinto de `Cliente` (webs propias) van en una sección aparte.
5. **Cachea** todas las respuestas en `cache/` para poder reanudar sin volver a descargar. Haz commit del avance cada 20 webs.
6. **Evidencia siempre:** cada hallazgo lleva la URL y el valor observado. Lo que no puedas comprobar se marca como `no verificado`. No inventes datos ni cifras.
7. Si la red del entorno no deja salir a internet, **avísame en cuanto lo detectes y para**. No intentes rodearlo.

## Fase 0: preparación (enséñame el resultado antes de seguir)
- Carga el CSV maestro y comprueba que cada `url_auditar` resuelve. Clasifica cada web:
  - OK;
  - redirige a otro dominio;
  - caída (5xx o timeout);
  - DNS no resuelve;
  - dominio aparcado o en venta;
  - página en construcción o modo mantenimiento.
- Enséñame una tabla con las webs que no están OK y el plan de trabajo con el tiempo estimado. Después **continúa sin esperar mi confirmación**.

## Fase 1: SEO (con un script de Python, asíncrono y con caché)
Para cada web:
1. **Acceso y seguridad:**
   - código HTTP y cadena de redirecciones (http→https, www o sin www: debe haber una sola versión canónica);
   - certificado SSL (validez y fecha de caducidad);
   - TTFB de la home.
2. **Indexabilidad:**
   - robots.txt: bloqueos, sitemap declarado y errores típicos (`Disallow: /`, wp-admin mal puesto);
   - `noindex` en meta robots o en la cabecera X-Robots-Tag;
   - canonical (autorreferente, apuntando a otro dominio, o a http);
   - sitemap: existe, nº de URLs, y en una muestra de 20 URLs, cuántas dan 4xx, redirigen o son noindex.
3. **On-page, por URL rastreada:**
   - title: falta, longitud, duplicados;
   - meta description;
   - H1: ninguno o varios;
   - nº de palabras;
   - imágenes: sin alt y las más pesadas;
   - enlaces internos rotos (muestra);
   - `html lang` y hreflang: muchas webs tienen gallego, español, inglés o portugués; comprueba que estén bien enlazadas entre sí.
4. **Datos estructurados** (extrae JSON-LD, Microdata y RDFa):
   - tipos presentes: Organization, LocalBusiness y sus subtipos, Product, FAQPage, Review/AggregateRating, BreadcrumbList, Article, Event;
   - errores evidentes;
   - que el NAP del schema coincida con el de la web.
5. **SEO local:**
   - NAP visible (nombre, dirección, teléfono) y coherente en toda la web;
   - mapa o enlace a la ficha de Google Business Profile;
   - página de contacto;
   - ciudad o zona en los titles;
   - páginas de servicio por localidad;
   - reseñas visibles.
6. **Contenido y E-E-A-T:**
   - blog o noticias (fecha del último post);
   - página "quiénes somos", autoría, testimonios, certificaciones;
   - contenido escaso o duplicado;
   - **restos de plantilla**: lorem ipsum, "Just another WordPress site", textos o imágenes demo, páginas de prueba indexadas, dominios de staging enlazados. Busca especialmente referencias a `clientes.ledmon.es` o `plantillaelementor`, porque ya apareció un caso con fuentes que se cargaban desde ahí.
7. **Tecnología:**
   - CMS y constructor: WordPress, Elementor, Divi, Shopify, Wix…;
   - plugin SEO: Yoast, Rank Math, SEOPress o ninguno;
   - plugin de caché;
   - plugins de Ledmon detectados;
   - banner de cookies: qué CMP es y si bloquea o retrasa la analítica.
8. **Medición.** Extrae del HTML los IDs `G-`, `GT-`, `GTM-`, `AW-` y el píxel de Meta. Compáralos con `ga_ids_medicion` y `gtm_contenedores` del CSV y clasifica cada web:
   - OK;
   - sin etiqueta;
   - ID distinto al que gestiona Ledmon;
   - contenedor duplicado;
   - Universal Analytics (UA-) todavía en el código.
9. **Rendimiento (niveles A y B).** PageSpeed Insights API v5, estrategia mobile, sobre la home:
   - puntuaciones de rendimiento, SEO, accesibilidad y buenas prácticas;
   - LCP, CLS, TBT e INP de campo (CrUX) si los hay.

   Usa `PSI_API_KEY` si existe esa variable de entorno. Sin clave, ve despacio para no quedarte sin cuota.

## Fase 2: GEO (preparación para buscadores con IA)
1. **Bots de IA en robots.txt.** Para GPTBot, OAI-SearchBot, ChatGPT-User, ClaudeBot, Claude-SearchBot, PerplexityBot, Google-Extended, Applebot-Extended, CCBot y Bingbot, indica si están permitidos o bloqueados.
2. **Bloqueo real por servidor o WAF.** Pide la home con el User-Agent de GPTBot, ClaudeBot y PerplexityBot y anota si devuelve 403, 429, un reto (challenge) o una respuesta distinta.
3. **llms.txt / llms-full.txt:** si existen y qué contienen.
4. **Contenido legible sin JavaScript:** si el texto principal y los datos clave (servicios, zona, teléfono, precios orientativos, horarios) están en el HTML que entrega el servidor.
5. **Entidad clara:**
   - Organization o LocalBusiness con `name`, `logo`, `address`, `telephone`, `areaServed` y `sameAs` (redes, Google Business Profile, directorios);
   - nombre comercial coherente.
6. **Contenido citable:**
   - respuestas directas a preguntas reales y FAQs;
   - datos concretos (precios orientativos, plazos, horarios, zonas, materiales);
   - listas y tablas;
   - páginas de servicio que dejen claro qué se ofrece, dónde y para quién;
   - fecha de actualización visible.
7. **Prueba de visibilidad (solo nivel A, 2–3 consultas por web).** Inventa búsquedas comerciales que haría un cliente real, como "taller mecánico en Cangas", "camping en Sanxenxo con piscina" o "clínica dental en Pontevedra". Haz cada una con WebSearch y anota:
   - si aparece el dominio y en qué posición aproximada;
   - qué competidores salen;
   - si hay respuestas o resúmenes que citen fuentes.

   Déjalo claro en el informe: es una **aproximación** y no la respuesta real de ChatGPT ni de AI Overviews.

## Fase 3: cruce con los datos de Google (del CSV)
- Webs con muchas impresiones y CTR bajo → mejorar titles y descriptions. Prioriza por impresiones.
- Webs con `sc_ultimo_dato` anterior al 27/09/2026 o sin impresiones → ¿web caída, desindexada o mal verificada?
- Webs con tráfico orgánico pero sin Analytics en Posiciona 1 → oportunidad de medición.
- Webs con `prioridad_medicion` Urgente o Arreglar → incluir el arreglo de la medición en su plan.

## Puntuación
- **Nota SEO y nota GEO de 0 a 100 por web**, con una rúbrica explícita: bloques, pesos y criterios documentados en el informe. Tiene que ser reproducible para repetirla cada trimestre.
- **Cada hallazgo lleva:**
  - severidad: crítica / alta / media / baja;
  - esfuerzo: S (<1 h), M (1 día) o L (>1 día);
  - responsable: Ledmon técnico, Ledmon contenido o el cliente;
  - **servicio de Ledmon asociado**.

## Entregables (carpeta `informe/`)
1. **`auditoria_ledmon.xlsx`**, con estas hojas:
   - **Resumen:** una fila por web, con nivel, tráfico, nota SEO, nota GEO, CMS, los 3 problemas principales, la oportunidad comercial y la prioridad.
   - **Hallazgos:** una fila por hallazgo, con dominio, categoría, severidad, URL de evidencia, valor observado, recomendación, esfuerzo, responsable y servicio.
   - Una hoja por bloque: Técnico, On-page, Schema, Local, GEO, Medición y Rendimiento.

   Usa filtros, colores por severidad y la primera fila fija.
2. **`informe_ejecutivo.md`** (máximo 4 páginas):
   - **Diagnóstico de la cartera:** los problemas que se repiten en muchas webs, con cuántas y cuáles.
   - **Las 10 acciones** de más impacto y menos esfuerzo.
   - **Lo que Ledmon puede desplegar en bloque** en todas las webs WordPress: plugin propio, ajustes de servidor o hosting, plantillas de schema local, llms.txt, revisión de robots…
   - **Propuesta de servicios y paquetes para vender** (SEO local, GEO, contenidos, WPO, medición y consentimiento), con a qué clientes ofrecer cada uno y por qué. Basado en sus datos, no en suposiciones.
3. **`clientes/<dominio>.md`:** una ficha de 1 página por web de nivel A y B, con:
   - estado y notas;
   - los 5 problemas clave con su evidencia;
   - quick wins;
   - plan a 30, 60 y 90 días;
   - un mensaje corto para el cliente, en español y sin tecnicismos, explicando qué proponemos y por qué.
4. **`plan_accion.csv`:** dominio, tarea, impacto (1–5), esfuerzo (S/M/L), prioridad, responsable y servicio. Debe poder importarse a un gestor de tareas.
5. **`scripts/`:** los scripts de rastreo y puntuación, documentados, para repetir la auditoría cada trimestre con el mismo CSV.

## Forma de trabajar
- Orden: Fase 0 → nivel A → B → C → D. Ve guardando los resultados por web, para que un corte no haga perder trabajo.
- Si una web falla, anótalo y sigue con la siguiente.
- Al terminar:
  - un resumen de 15 líneas en el chat;
  - la lista de lo que no se pudo verificar;
  - qué datos extra mejorarían la auditoría (por ejemplo, las consultas y páginas de Search Console de cada web, o acceso a Google Business Profile).
