# Plan de mejoras · versión móvil del informe de cartera

Base revisada: `MorosidadSectorSolidario_Movil.Report.zip` (9 nov. 2026).
Este documento cubre la **fase 1 (contextualización)** y la **fase 2 (orden por impacto y dificultad)**.
El plan paso a paso de cada requisito (fase 3) se hace después, requisito por requisito.

## Hallazgos del zip que afectan a todo el plan
- **Hay dos modelos.** El informe móvil apunta a su propia copia, `MorosidadSectorSolidario_Movil.SemanticModel`, y no al modelo del de escritorio. Las diferencias son pequeñas: los saldos tienen 2 decimales y hay cambios de columnas ocultas. Pero cada medida nueva (ranking, fechas, cobertura) habría que crearla dos veces.
- **Ya quitaste los selectores de rango de fechas** en Por entidad y Cobertura. Siguen los segmentadores de **mes** ("Nombre Mes") en las 5 páginas, con "Agosto" fijo.
- **Inicio ya no tiene "Entidades a comparar".** Solo quedan Mes y Su entidad.
- **Sincronización actual:**
  - Mes está sincronizado en las 5 páginas.
  - Su entidad (`dm_EntidadSeleccionada`) está sincronizado en Inicio, Por entidades, Por entidad y Cobertura.
  - Entidades a comparar (`dm_Entidades`), Línea, Tipo de entidad, Departamento y Municipio no están sincronizados.
- **Los datos vienen de un flujo de datos de Power BI** (`PowerPlatform.Dataflows`) de MDA, no directamente de la Supersolidaria.

---

## Fase 1 · Contextualización de cada requisito

| # | Requisito | Qué significa | Cómo se hace en Power BI | Riesgo o duda |
|---|---|---|---|---|
| 1 | Filtros ocultos en Inicio | Inicio muestra solo los botones. El filtro aparece cuando el usuario elige un informe, por ejemplo el de Su entidad al tocar "Informe por entidad". | Marcadores (bookmarks) que muestran u ocultan grupos de visuales, más botones. Los marcadores funcionan en "Publicar en la web". | Si se aplica el #11, el filtro de mes desaparece y queda solo Entidad. Eso simplifica mucho este punto. |
| 2 | Definición de cada indicador | El usuario ve la fórmula y el significado de cada indicador: tasa de mora, tasa de riesgo, riesgo/mora, cobertura, cobertura C-D-E y deterioro/cartera. | Un botón "i" por tarjeta o gráfica que abre un panel con la definición (marcadores), más una página "Glosario". El manual de marca lo llama *pophover*. Las definiciones ya están en `docs/diccionario-de-datos.md`. | En el celular no hay "pasar el mouse": la ayuda debe abrirse al tocar. Son muchos marcadores si se pone un panel por visual. |
| 3 | Tooltips solo al tocar y con regla | La información emergente de las barras aparece solo al tocar, y aparece o no según una condición. | En el celular el tooltip ya aparece solo al tocar, porque no existe el hover. La "regla" se logra con medidas que devuelven vacío y ocultan el contenido. La alternativa es una página de detalle (*drill-through*) que se abre al tocar la barra. | **Falta definir la regla.** Power BI no permite encender o apagar un tooltip con una condición; solo se puede vaciar su contenido. |
| 4 | Ranking en benchmarking | Mostrar "Posición 12 de 180" de la entidad en cada indicador. | Medidas DAX con `RANKX` sobre las entidades, en tarjetas y en el tooltip. | **Falta definir:** ¿ranking contra todo el sector o contra las entidades elegidas para comparar? ¿Por línea o total? ¿Menor mora = puesto 1? Las entidades con datos mal reportados, como COOTRAUNION, distorsionan el ranking. |
| 5 | Selector en la gráfica de cobertura | La línea histórica de Cobertura deja elegir entre cobertura total, cobertura individual y cobertura C-D-E. | Un parámetro de campos con 3 medidas, el mismo patrón de "Indicadores gráfico de líneas". Falta crear la medida "Cobertura individual" (deterioro individual ÷ cartera en riesgo). | Bajo. |
| 6 | Sincronizar filtros entre páginas | El filtro que el usuario elige en una página sigue en las demás. | Grupos de sincronización (*sync groups*) en los segmentadores. Mes y Su entidad ya están; faltan Entidades a comparar, Línea y los filtros de Por sector. | Hay que decidir qué filtros deben viajar entre páginas. Por ejemplo, Departamento no tiene sentido en Por entidad. Depende del #11. |
| 7 | Mostrar tres fechas | (a) último corte de los datos, (b) última actualización hecha por MDA, (c) última publicación de la Supersolidaria. | (a) `MAX(Fecha reporte)`. (b) Una consulta que guarda `DateTime.LocalNow()` en cada actualización, ajustada a la hora de Colombia. (c) Leer la fecha de la página "Entidades vigiladas que reportan información", o escribirla a mano. | (c): leer la página web desde Power Query puede romperse si la Supersolidaria cambia su página. Hay que confirmar la URL exacta y si el servicio la puede actualizar sin puerta de enlace. |
| 8 | Histórico de actualizaciones | Una página con la bitácora: cada actualización con su fecha, el corte de datos incluido y la fecha de la Supersolidaria, más una explicación del proceso y el enlace a la fuente. | Power BI **no guarda historia**: cada actualización reemplaza los datos. Se necesita una tabla persistente: una lista de SharePoint o un Excel que MDA mantiene, o el flujo de datos con actualización incremental. | Es el más complejo. Hay que decidir dónde vive la bitácora y quién la llena. |
| 9 | KPIs de cobertura de a 2 por fila | En Cobertura, las tarjetas en filas de 2. | Cambiar la cuadrícula de las tarjetas. | Hay que decidir si la tarjeta oscura sigue sola arriba o entra en la cuadrícula 2×2. |
| 10 | Tooltips en verde | Las páginas Tooltip-Mora y Tooltip-Riesgo aún tienen los colores azules de escritorio. | Cambiar el fondo, los encabezados y los textos a la paleta Monitor GRC. | Bajo. |
| 11 | Quitar el selector de fecha | El informe muestra siempre el último corte; el usuario no elige la fecha, solo se le informa cuál es. | Una columna `Es último corte` en Calendario, que se recalcula en cada actualización, aplicada como filtro a los visuales del mes actual. Las gráficas históricas se excluyen de ese filtro. Se quitan los segmentadores de mes. | **Alto impacto en la exactitud:** hoy "Agosto" está fijo, y al llegar septiembre alguien tendría que cambiarlo a mano en cada página. Sin el filtro, los saldos de varios meses se sumarían. Toca medidas y todas las páginas. |

---

## Fase 2 · Orden por impacto y dificultad

Escala de 1 (bajo) a 5 (alto).

| # | Requisito | Impacto | Dificultad | Depende de |
|---|---|:-:|:-:|---|
| 11 | Quitar el selector de fecha (siempre el último corte) | 5 | 3 | P0 |
| 7 | Mostrar las tres fechas (a y b; c después) | 4 | 2 | 11 |
| 6 | Sincronizar filtros | 4 | 2 | 11 |
| 4 | Ranking en benchmarking | 4 | 3 | P0 y definiciones |
| 2 | Definición de indicadores | 4 | 3 | — |
| 5 | Selector en la gráfica de cobertura | 3 | 2 | P0 |
| 1 | Filtros ocultos en Inicio | 3 | 3 | 11 y 6 |
| 8 | Histórico de actualizaciones | 3 | 5 | 7 y decisión de almacenamiento |
| 10 | Tooltips en verde | 2 | 1 | — |
| 9 | KPIs de a 2 por fila | 2 | 1 | — |
| 3 | Tooltips al tocar con regla | 2 | 4 | definir la regla |

### Orden propuesto (olas)
0. **P0 · Una sola base:** decidir si el móvil usa el modelo compartido o su copia, y unificar. Sin esto, cada medida nueva se hace dos veces.
1. **Ola 1 · Ajustes rápidos:** #10, #9 y #5. Son poco riesgosos y se ven de inmediato.
2. **Ola 2 · Cimientos:** #11, luego #7 (a y b), luego #6. El #11 cambia cómo funciona el filtro de fecha en todo el informe, así que va antes de sincronizar y de reorganizar Inicio.
3. **Ola 3 · Valor analítico:** #4 y #2.
4. **Ola 4 · Experiencia:** #1 y #3.
5. **Ola 5 · Trazabilidad:** #8 y #7 (c).

### Decisiones que necesito antes del paso a paso
1. **P0:** ¿un solo modelo para escritorio y móvil, o modelos separados?
2. **#4:** ¿ranking contra todo el sector o contra las entidades comparadas? ¿Total o por línea? ¿Se excluyen entidades con datos dudosos?
3. **#3:** ¿cuál es la "regla" para mostrar u ocultar el tooltip?
4. **#6:** ¿qué filtros deben viajar entre páginas?
5. **#7c y #8:** ¿la fecha de la Supersolidaria se lee automáticamente de su página o se escribe a mano? ¿Dónde vive la bitácora (SharePoint, Excel o el flujo de datos)?
6. **#9:** ¿la tarjeta oscura de cobertura queda sola arriba, o se pasa a una cuadrícula 2×2?

---

## Decisiones del usuario (9 nov. 2026)
1. **Modelo:** por ahora el móvil usa su propia copia (`MorosidadSectorSolidario_Movil.SemanticModel`). Se unificarán después.
2. **Ranking:** va en Por entidades, para tasa de mora, tasa de riesgo y riesgo/mora. Además, una gráfica nueva de saldo de cartera total y saldo en mora. Debajo de cada gráfica va una frase del tipo "COOPRUDEA es la número 10 entre 100 entidades en …".
3. **Tooltip:** en Por entidad, al tocar una línea de crédito en "Tasa de mora" o "Tasa de riesgo", aparece debajo el detalle (el contenido de Tooltip-Mora o Tooltip-Riesgo). Si no hay selección, el recuadro sale vacío e invita a tocar una línea.
4. **Sincronización:** todos los filtros. La fecha deja de ser un filtro: siempre se usa el último corte y el informe lo muestra.
5. **Fechas:** solo la fecha que publica la Supersolidaria en su página. No se hace la bitácora (#8 queda fuera por ahora).
6. **Cobertura:** cuadrícula de KPIs, y el selector de la línea de tiempo usa los mismos indicadores de la cuadrícula.

## Fase 3 · Plan paso a paso (todo sobre el modelo y el informe móvil)

### Ola 1 · Ajustes rápidos
**#10 Tooltips en verde**
1. En Tooltip-Mora y Tooltip-Riesgo: fondo de página blanco, encabezados de la tabla en `#1B4B3D` con texto blanco, filas alternas en `#F3F5F4`, textos en `#252423` y la tarjeta del título en `#008282`.
2. Verificar en Desktop, pasando el cursor sobre las barras de Por sector.

**#9 + #5 Cobertura: cuadrícula de KPIs y selector de indicador**
1. Medida nueva `Cobertura individual` = Deterioro individual ÷ Saldo en riesgo, más su versión "con entidad seleccionada".
2. Tarjetas en filas de 2: Cobertura total · Cobertura individual / Cobertura C-D-E · Deterioro sobre cartera bruta (la distribución exacta se confirma; ver dudas).
3. Parámetro de campos nuevo `Indicador cobertura` con los indicadores de la cuadrícula. Su segmentador va encima de la línea de tiempo, que pasa a usar ese parámetro.

### Ola 2 · Cimientos
**#11 Siempre el último corte**
1. Columna calculada `Calendario[Es último corte]` = la fecha del mes es igual a `MAX(tb_Balance[Fecha reporte])`. Se recalcula en cada actualización.
2. Filtro de página `Es último corte = Verdadero` en Inicio y Por entidades. En Por sector, Por entidad y Cobertura, el filtro va a nivel de visual: todos excepto las líneas de tiempo, que muestran el histórico desde octubre de 2025.
3. Quitar los segmentadores de mes de las 5 páginas.
4. Revisar las medidas "sin filtros" y los tooltips, para que tampoco sumen meses.
5. Validar: los valores de agosto de 2026 deben coincidir con `docs/validacion-medidas.md`.

**#7 Fechas visibles**
1. Medida `Fecha de corte` = `MAX(Fecha reporte)`, con formato "agosto de 2026". Va en el encabezado de cada página ("Datos a agosto de 2026").
2. Fecha de la Supersolidaria: una consulta de Power Query lee la página pública y extrae "Fecha de actualización". Se guarda en una tabla de una fila y se muestra en Inicio. Requiere la URL exacta (ver dudas).
3. Probar la actualización en el servicio: si la página no se puede leer sin puerta de enlace, se reemplaza por una tabla escrita a mano.

**#6 Sincronizar todos los filtros**
1. Grupos de sincronización por campo: Su entidad (`dm_EntidadSeleccionada[SIGLA]`), Entidades a comparar (`dm_Entidades[SIGLA]`), Línea, Tipo de entidad, Departamento, Municipio y "Ver por".
2. En las páginas donde un filtro no se ve, agregar un segmentador oculto sincronizado, para que el filtro se aplique igual.
3. Revisar el efecto cruzado: por ejemplo, filtrar Departamento en Por sector también reduce las entidades de Por entidades. Mostrar un aviso de filtros activos.

### Ola 3 · Valor analítico
**#4 Ranking en Por entidades**
1. Medidas de ranking con `RANKX` sobre todas las entidades con datos en el último corte, para la entidad de "Su entidad":
   - tasa de mora y tasa de riesgo: menor es mejor (puesto 1 = la de menor tasa);
   - riesgo/mora;
   - saldo total y saldo en mora: mayor = puesto 1.
2. Medida `Total entidades` = número de entidades con cartera en el último corte.
3. Medidas de texto: "COMEDAL es la número 34 entre 180 entidades en tasa de mora (1 = la más baja)". Van en un cuadro de texto debajo de cada gráfica.
4. Gráfica nueva: barras de saldo de cartera total y saldo en mora por entidad comparada, con su frase de ranking.
5. Validar el ranking en Python con los mismos datos.

**#2 Definición de indicadores**
1. Página nueva "Glosario", con fórmula, explicación en lenguaje sencillo y ejemplo de cada indicador. Se toma de `docs/diccionario-de-datos.md`.
2. Un botón "i" junto al título de cada tarjeta o gráfica que lleva al Glosario, y un botón "Volver".

### Ola 4 · Experiencia
**#3 Detalle debajo de las barras en Por entidad**
1. Debajo de "Tasa de mora por línea" y de "Tasa de riesgo por línea" va una tabla con el mismo contenido de Tooltip-Mora o Tooltip-Riesgo (franjas de mora o calificación A-E).
2. Interacciones: cada barra filtra solo su tabla de detalle.
3. Una medida que detecta si hay una línea seleccionada. Sin selección, la tabla queda vacía y un texto dice "Toca una línea de crédito en la gráfica de arriba para ver el detalle".
4. Desactivar el tooltip emergente en esas dos gráficas, para no duplicarlo.

**#1 Filtros ocultos en Inicio**
1. Sin el mes, en Inicio quedan Su entidad y Entidades a comparar.
2. Los filtros se ocultan con marcadores. Al tocar "Informe por entidad", "Cobertura" o "Benchmarking" aparece el filtro necesario, con un botón "Ver informe". "Indicadores por sector" va directo, sin filtro.
3. Probar que los marcadores no reinicien los filtros (marcadores sin "Datos", solo "Visualización").

### Fuera de alcance por ahora
- **#8** bitácora de actualizaciones.
- La unificación de los modelos de escritorio y móvil.

## Decisiones del usuario (9 nov. 2026, segunda ronda)
- **Cobertura (#9 y #5):**
  - Cuadrícula de 2 por fila, sin tarjeta oscura: Cobertura total, Cobertura individual, Deterioro / cartera bruta y Deterioro total.
  - La línea de tiempo usa un parámetro de campo nuevo con todos los indicadores de cobertura, incluido Deterioro total en pesos, como el parámetro de Por entidad (que mezcla pesos y tasas).
  - Pendiente: decidir si Cobertura C, D y E sigue en las tarjetas o queda solo en el selector.
- **Ranking (#4):**
  - Solo en la página Por entidades (Benchmarking); no en Por sector.
  - El universo son las entidades que cumplen los filtros activos de la página. Por ejemplo, con Departamento = Antioquia, el ranking es entre las cooperativas de Antioquia, no del total. En DAX, `RANKX` sobre `ALLSELECTED(dm_Entidades[SIGLA])` en lugar de `ALL`.
  - En las tasas, el puesto 1 es la de menor valor (la mejor). En los saldos, el puesto 1 es la de mayor valor.
  - No se excluye ninguna entidad (tampoco COOTRAUNION): cuando la entidad corrija su reporte, no habrá que deshacer una regla.
- **Fecha de la Supersolidaria (#7):** se toma de https://www.supersolidaria.gov.co/es/content/entidades-vigiladas-que-reportan-informacion, el enlace que ya está en el informe. Desde este entorno no se pudo abrir la página: el proxy rechaza el dominio. Hay que confirmar en Power BI Desktop dónde aparece la fecha y si Power Query la puede leer.

## Avance
- **Ola 1 implementada (9 nov. 2026)** sobre la versión móvil del usuario:
  - Medidas nuevas: `Cobertura individual` y `Cobertura individual con entidad seleccionada`.
  - Tabla nueva `Indicador cobertura` (parámetro de campo con 7 indicadores).
  - Cobertura: una sola tarjeta 2×2. Fila 1: Cobertura individual y Provisión individual. Fila 2: Cobertura total y Provisión total.
  - Un selector de indicador sobre la línea de tiempo.
  - Tooltip-Mora y Tooltip-Riesgo pasan a la paleta verde.
  - Valores esperados para COMEDAL a agosto de 2026: cobertura individual 60,28 %, provisión individual $13.962.982.499, cobertura total 107,90 %, provisión total $24.992.772.751.

## Integración (10 nov. 2026)
Se unieron los 4 agentes en la rama y se aplicaron las decisiones finales:
- Cobertura: 7 indicadores, cada uno en su tarjeta (6 en matriz 2×3 + Provisión total a lo ancho). El selector de la línea de tiempo mantiene los 7.
- Glosario: se dejan los ejemplos de agosto de 2026 (con nota) para poder explicarlos.
- Ranking Riesgo/Mora: se mantiene con "1 = la más baja", como las tasas.
- Botones "i" regenerados: uno por gráfica, recolocados tras mover Por entidades (ranking) y Por entidad (detalle).
Validado: JSON correcto, sin visuales fuera de página, sin interacciones rotas, 65 medidas sin duplicados.
Pendientes: #1 filtros ocultos en Inicio; #7 fecha de la Supersolidaria (consulta lista, falta validar en vivo); #8 fuera de alcance.

## #1 Filtros ocultos en Inicio (11 nov. 2026)
Implementado con páginas de filtro separadas (navegación por botones), no con marcadores, por ser más confiable en el enlace publicado.
- Inicio: se quitan los segmentadores visibles; quedan el encabezado (con fecha de corte y Supersolidaria) y 4 botones.
- "Indicadores por sector" navega directo a Por sector (no necesita filtros).
- "Benchmarking", "Informe por entidad" y "Cobertura" abren cada uno su página de filtros (oculta en el informe publicado), con botón "Volver".
- Benchmarking-filtros: paso 1 "Entidades a comparar" (lista, selección múltiple) y paso 2 "Su entidad" (lista, selección única) limitada a las elegidas en el paso 1 mediante la medida `Entidad está en comparadas`.
- Entidad-filtros y Cobertura-filtros: solo "Su entidad" (lista); "Entidades a comparar" aparece como recuadro bloqueado "No aplica en este informe".
- Los dos filtros de entidad usan el mismo estilo de lista (listSlicer). Sincronizados: Su entidad (grupo SIGLA), Comparar (grupo SIGLA1), por lo que la selección viaja a las páginas de informe.
- Por revisar en Desktop: activar "Buscar" en cada lista; que la selección única se vea bien en el listSlicer; y, sobre todo, que el filtro del paso 2 (subconjunto del paso 1) funcione.
