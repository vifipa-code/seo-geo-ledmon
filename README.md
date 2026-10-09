# Ledmon — datos para la auditoría SEO + GEO de clientes

Inventario de todas las webs que Ledmon gestiona en Google: Analytics, Search Console y Tag Manager de la cuenta Posiciona 1, más las cuentas de Analytics que se pasaron a Posiciona 2. Datos sacados por API el **07/10/2026**, con una ventana de **90 días** (08/07/2026–06/10/2026), y actualizados con los cambios hechos del 07 al 09/10/2026.

Todos los CSV usan `;` como separador y UTF-8 con BOM, así que se abren tal cual en Excel.

## `datos/clientes_ledmon.csv` — listado maestro (una fila por dominio, 202 webs)

| Columna | Qué es |
|---|---|
| dominio | Dominio raíz, sin www. Es la clave de unión con los otros CSV. |
| url_auditar | URL de inicio recomendada: la versión https con más clics en Search Console. |
| cliente | Nombre comercial (de Tag Manager o Analytics). |
| tipo | `Cliente`, o web propia de Ledmon o de Víctor. |
| nivel_trafico | A = ≥1.000 usuarios GA o ≥500 clics SC en 90 días · B = ≥200 usuarios o ≥100 clics · C = algo de actividad · D = sin datos. |
| cliente_activo | **Vacío. Rellenar con `no` para que la auditoría salte esa web.** |
| cuenta_google | Dónde está su Analytics: Posiciona 1, Posiciona 2, o sin Analytics en Posiciona 1 (puede estar en la cuenta del cliente). |
| ga_cuentas / ga_ids_medicion / ga_usuarios_90d | Cuentas, IDs G- y usuarios activos en 90 días (suma de propiedades). |
| gtm_contenedores / gtm_estado | Contenedores GTM-, con prioridad y diagnóstico de la revisión del 07/10. |
| sc_propiedades / sc_permiso | Propiedades de Search Console y permiso de Posiciona 1 (`Sin verificar` = ya no ve datos). |
| sc_clics_90d / sc_impresiones_90d / sc_ctr_90d_% | Rendimiento orgánico. Si hay varias propiedades, se toma la de más clics. |
| sc_ultimo_dato | Último día con datos en Search Console. |
| prioridad_medicion | La peor prioridad de su medición: Urgente / Arreglar / Revisar / Limpieza / Orden / OK. |
| hosts_vistos | Todos los hostnames vistos en las tres herramientas. |
| notas | Contexto manual. |

## Listados en bruto (detalle por fuente)

- `datos/analytics_propiedades.csv`: una fila por propiedad GA4 en Posiciona 1, con ID de medición, usuarios en 90 días y hostnames del flujo web.
- `datos/tagmanager_contenedores.csv`: un contenedor por fila. Incluye la versión publicada, el nº de etiquetas, los tipos, los IDs que envía, si su GA4 recibe datos, y el diagnóstico (prioridad / estado / recomendación).
- `datos/search_console_propiedades.csv`: una propiedad por fila (dominio o prefijo URL). Incluye el permiso, los clics, las impresiones, el último dato y el diagnóstico.

## Avisos

- 100 webs aparecen "sin Analytics en Posiciona 1". Muchas solo están en Search Console: su Analytics puede estar en la cuenta del propio cliente o no existir.
- Algunas webs ya no son clientes. Márcalas en `cliente_activo` antes de lanzar la auditoría, o deja que la auditoría las incluya y fíltralas después.
- Las cuentas que se pasaron a Posiciona 2 (11 webs) no tenían datos en Analytics, pero varias tienen tráfico orgánico real en Search Console.
