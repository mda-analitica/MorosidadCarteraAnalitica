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
