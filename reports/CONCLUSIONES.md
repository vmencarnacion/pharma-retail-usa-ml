# Conclusiones del estudio

> **Advertencia:** este informe describe una corrida del generador con semilla 42 y `--base-records-per-month 6200` (el valor por defecto). Es una muestra **sintetica** con fines academicos. No son ventas reales del mercado de Estados Unidos, no hay representatividad estadistica y no se puede usar para afirmar causalidad de la pandemia de COVID-19 sobre ningun medicamento.

## Pregunta de investigacion

¿Como varia la demanda modelada (sintetica) de 62 productos farmaceuticos a lo largo de 94 meses (enero 2019 - octubre 2026), y cuanto de esa variacion viene de una calibracion documentada por categoria versus ruido aleatorio del generador?

## Hallazgo 1: evolucion por categoria

![Trayectoria por grupo](figures/01_tendencias_por_categoria.png)

Comparando el promedio de unidades modeladas por producto-mes en 3 periodos de distinta duracion (2019 pre-pandemia: 14 meses: Mar 2020-May 2023: 39 meses: Jun 2023-Oct 2026: 41 meses), el grupo **CONTROLLED** (los 2 opioides, DEA Schedule II) es el unico que cae y no se recupera: pasa de ~5,806 unidades/producto-mes en 2019 a ~4,684 en la ultima fase, una baja de casi 20%. Los grupos **OTC** y **RX** se mantienen practicamente planos en las 3 fases (entre 5,745 y 5,844). Esta caida sostenida de CONTROLLED viene de la calibracion `DOCUMENTED_CATEGORY_PROXY` (el indice de dispensacion de opioides del CDC), no es casualidad del ruido sintetico.

## Hallazgo 2: variaciones por medicamento (2020 vs 2019)

![Variacion por medicamento](figures/02_cambios_por_producto_2020.png)

Comparando los promedios de meses completos de 2019 contra 2020, los 5 productos con mayor baja son todos antibioticos RX: Doxycycline (-24.2%), Amoxicillin (-21.8%), Macrobid (-21.8%), Zithromax (-21.7%) y Cefdinir (-17.6%). Estos 5 productos son justo los que tienen calibracion `DOCUMENTED_CATEGORY_PROXY` basada en un estudio de prescripcion de antibioticos ambulatorios, asi que esa baja esta **documentada por categoria**, no es ruido. En cambio, los productos que mas suben (Advil +5.6%, Norvasc +5.1%, Eliquis +4.8%, Afrin +4.3%, Xarelto +4.0%) no superan el 6% y son una mezcla de OTC y RX sin ningun patron de categoria en comun: esa variacion si es puro ruido sintetico del generador (`synthetic_monthly_product_noise_sd`).

## Hallazgo 3: cobertura y calidad de la evidencia

![Niveles de calibracion](figures/03_niveles_calibracion.png)

De las 5,828 filas producto-mes, el **92.7%** (5,400 filas) son `SYNTHETIC_NEUTRAL_NO_PRODUCT_EVIDENCE`: multiplicador anual = 1, solo varian por ruido aleatorio y corresponden a los 55 productos sin ninguna fuente detras. Solo el **6.6%** (384 filas, los 2 opioides + 5 antibioticos) tiene un proxy documentado (`DOCUMENTED_CATEGORY_PROXY`). El **0.75%** restante (44 filas) es `CARRY_FORWARD_CATEGORY_PROXY_SYNTHETIC`: los meses de 2025-2026 de los opioides, donde se repite el ultimo dato documentado de 2024 porque todavia no hay cifras mas recientes del CDC. En otras palabras: solo 7 de los 62 productos (11.3%) tienen algun tipo de respaldo documental; los otros 55 son pura simulacion neutra.

## Verificacion adicional: tamanio de muestra vs volumen modelado

Corri el generador dos veces con la misma semilla (42) pero cambiando `--base-records-per-month` de 200 a 6,200. El numero de eventos de muestra cambia de 18,552 a 575,028 (31 veces mas), pero el total de unidades modeladas se queda exactamente igual: 33,639,168 en ambos casos. Esto confirma que ese parametro solo reparte el mismo volumen modelado en mas o menos eventos sinteticos; no cambia la demanda que se esta estudiando.

## Observaciones numericas verificables

1. El dataset analitico tiene 5,828 filas (62 productos x 94 meses), confirmado por los tests de integridad.
2. El total de unidades abstractas modeladas en todo el periodo es 33,639,168.
3. Octubre de 2026 es un mes parcial: su `coverage_factor` es 0.613 (19 de 31 dias), por eso no se compara su total crudo contra un mes completo.
4. El grupo CONTROLLED cae de un promedio de 5,873.2 unidades/producto-mes en 2019 a 4,638.9 en la fase Jun 2023-Oct 2026 (sobre meses completos).
5. Cambiar `--base-records-per-month` de 200 a 6,200 multiplica los eventos de muestra por 31 pero no cambia ni una unidad del volumen modelado total (33,639,168 en ambos casos).

## Distincion: observacion de la simulacion vs hecho respaldado por fuentes externas

- **Respaldado por fuente externa (documentado):** la tendencia a la baja de los opioides (CONTROLLED) sigue el indice de dispensacion del CDC, y la baja de los 5 antibioticos sigue un estudio de prescripcion ambulatoria. Son proxies de **categoria**, no series individuales verificadas por producto.
- **Observacion pura de la simulacion (sin fuente):** las variaciones mes a mes de los otros 55 productos, y el "dentado" que se ve en los graficos de OTC y RX, vienen del ruido aleatorio del generador. No reflejan ningun evento real de mercado.
- **Importante:** aunque 7 productos tengan un proxy documentado a nivel de categoria, eso no equivale a tener series de ventas reales verificadas producto por producto; sigue siendo una calibracion aplicada por igual a todos los productos del grupo, mes a mes.

## Limitaciones

- Muestra sintetica no probabilistica: no representa el mercado real de EE. UU. ni permite ranking real de medicamentos mas vendidos (todos arrancan con la misma referencia artificial de 5,850 unidades/mes).
- Las 33,639,168 unidades modeladas son abstractas: no son tabletas, dosis, pacientes ni dolares.
- Los proxies de categoria se aplican de forma uniforme a todos los meses del año y a todos los productos del grupo, sin diferenciar por producto individual.
- No hay precios, geografia estatal, ni causa clinica documentada en el dataset.
- Una diferencia 2020 vs 2019 en estos datos sinteticos no prueba un efecto causal real del COVID-19; solo refleja el supuesto que se metio al generador.
- El catalogo de productos es una preseleccion: los NDC y fechas de comercializacion individuales no estan validados de forma independiente.

## Proximos pasos

- Conseguir calibraciones documentadas por producto individual (no solo por categoria) para mas medicamentos.
- Validar los NDC y fechas de comercializacion contra la base de la FDA antes de cualquier afirmacion regulatoria.
- Si se quisiera usar este proyecto con fines de ML (Fase 7 del mandato), probar deteccion de cambio estructural en la serie de CONTROLLED, ya que es el unico grupo con una tendencia clara y documentada.
- Documentar cualquier cambio de semilla o parametros en `metadata_manifest.json` para que el analisis siga siendo reproducible.
