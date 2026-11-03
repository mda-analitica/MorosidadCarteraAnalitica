# Diccionario de datos y lógica: MorosidadSectorSolidario

Referencia técnica del modelo semántico y del informe, escrita para consulta futura (por el dueño y por Claude).
Cubre tablas, columnas, relaciones, catálogo de cuentas, medidas (con su lógica y dónde se usan), parámetros de campos y páginas.
Complementa a `docs/conocimiento-informe.md`, que guarda el contexto de negocio, las decisiones y los pendientes.

Última actualización: 2026-11-03. Fuente: archivos del proyecto PBIP y lectura del caché `cache.abf` (datos hasta diciembre de 2025).

---

## 1. Flujo de datos

1. **Fuente pública:** la Superintendencia de la Economía Solidaria publica la información financiera de las entidades vigiladas en https://www.supersolidaria.gov.co/es/content/entidades-vigiladas-que-reportan-informacion (confirmado por el dueño).
2. **Dataflows de Power BI Service:** los datos se cargan y limpian en dataflows y de ahí pasan al modelo semántico. El código de esos dataflows no está en este repo.
3. **Modelo semántico** (`MorosidadSectorSolidario.SemanticModel`, cultura `es-CO`, modo Import).
4. **Informe** (`MorosidadSectorSolidario.Report`): 4 páginas visibles y 2 tooltips ocultos, publicado en Power BI Service y embebido en un iframe público (sin inicio de sesión).

Alcance actual (confirmado): solo cooperativas de ahorro y crédito, y solo desde 2025-01-01, porque desde esa fecha hay información de cartera en mora.

| Tabla del modelo | Origen |
|---|---|
| `tb_Balance` | Dataflow `210af2db-4205-47ea-b3cf-a4b57747d9f8` (workspace `ca53e802-ef7f-4429-b643-fe316dbbff4b`), entidad `tb_Balance` |
| `dm_Entidades` | Dataflow `cc8c038e-23dd-4a57-97cd-4bfe5e79b796` (mismo workspace), entidad `Entidades_PUC` |
| `Calendario` | Dataflow `3103eda1-7a05-4d3d-8016-d3dcdd60938c` (workspace `7d7a12b8-9d3c-48ad-ab9b-018ceb413f07`), entidad `Calendario` |
| `dm_Cuentas` | Tabla escrita a mano dentro del modelo (datos incrustados) |
| `dm_CuentasIndice` | Consulta auxiliar escrita a mano (solo se usa para armar `dm_Cuentas`; no se carga como tabla) |
| `dm_EntidadSeleccionada` | Copia de `dm_Entidades` (referencia de la consulta) |
| `Medidas` | Tabla vacía que solo contiene las medidas |
| `Parámetro`, `Parámetro 2`, `Indicadores gráfico de líneas` | Parámetros de campos (tablas calculadas) |

## 2. Tablas y columnas

### `tb_Balance` (hechos)
Una fila por fecha, entidad y cuenta del PUC. 4.335.629 filas en el caché (enero a diciembre de 2025).

| Columna | Tipo | Descripción |
|---|---|---|
| `Fecha reporte` | Fecha | Cierre de mes del reporte (último día del mes). Se filtra `>= 2025-01-01`. |
| `CODIGO ENTIDAD` | Texto | Código de la entidad en la Supersolidaria. Une con `dm_Entidades`. |
| `CODRENGLON` | Texto | Código de cuenta del PUC (renglón). Une con `dm_Cuentas`. |
| `VALOR EN PESOS` | Entero | Saldo de la cuenta en pesos colombianos. Puede ser negativo. |

Lógica de carga: se filtran fechas desde 2025-01-01, se cruza con `dm_Entidades` y se descartan las filas cuya entidad no está allí (así solo quedan las entidades del alcance).

### `dm_Entidades` (dimensión)
174 entidades. Tipos: 140 Especializada de ahorro y crédito, 32 Multiactiva con ahorro y crédito, 2 Integral con ahorro y crédito. `CODIGO ENTIDAD` es único. Las entidades `1271` y `1376` no tienen balance.

| Columna | Tipo | Descripción |
|---|---|---|
| `CODIGO ENTIDAD` | Texto | Llave de la entidad. |
| `TIPO ENTIDAD SOLIDARIA` | Texto | Tipo de entidad. Solo pasan los tres tipos de arriba. |
| `MUNICIPIO` | Texto | Municipio del domicilio. |
| `DEPARTAMENTO` | Texto | Departamento del domicilio. |
| `SIGLA` | Texto | Sigla de la entidad. Si viene vacía se usa el nombre completo. Es lo que se muestra en los gráficos y segmentadores. |
| `Filtros todos` | Texto (calculada) | Constante `"Sin filtros"`. Sirve como opción "sin dimensión" del parámetro `Parámetro 2`. |

### `dm_EntidadSeleccionada`
Copia de `dm_Entidades` con las columnas `CODIGO ENTIDAD`, `SIGLA`, `TIPO ENTIDAD SOLIDARIA`, `MUNICIPIO` y `DEPARTAMENTO`. Se usa en segmentadores para que el usuario elija **la** entidad que quiere analizar sin filtrar al resto (para poder compararla). Por decisión del dueño, la entidad la elige siempre el usuario (no hay valor por defecto).

### `dm_Cuentas` (catálogo de cuentas)
75 cuentas del PUC que se usan para calcular los indicadores. Ver catálogo completo en la sección 4.

| Columna | Tipo | Descripción |
|---|---|---|
| `CODRENGLON` | Texto | Código de cuenta. Se ordena por `Índice`. |
| `Descripción` | Texto | Texto de la cuenta, tal como viene en el PUC. |
| `Línea` | Texto | Línea de crédito: Vivienda, Consumo con libranza, Consumo sin libranza, Microcrédito, Comercial. |
| `Estado` | Texto | `VERDADERO` si la cuenta cuenta para el indicador (ver sección 4); `FALSO` si no. |
| `Tipo` | Texto | `Riesgo` (cuentas 14xx, cartera por categoría) o `Mora` (cuentas 84xx, cartera por rango de días). |
| `Índice` | Entero | Orden de aparición en el catálogo. |
| `Altura` | Texto | En `Riesgo`: categoría A a E. En `Mora`: rango de días de mora. |

### `Calendario`
Fechas diarias (2.190 en el caché, hasta 2028). Filtro: `Fecha > 2025-01-01`. Las columnas son `Fecha`, `Año` (texto), `Mes` (número) y `Nombre Mes` (ordenado por `Mes`). Tiene una variación de jerarquía de fechas automática (`LocalDateTable_…`).

### Parámetros de campos
| Tabla | Opciones | Uso |
|---|---|---|
| `Parámetro` | Riesgo/Mora, Saldo en mora, Saldo en riesgo, Saldo total, Tasa mora, Tasa riesgo (todas "con entidad seleccionada") | Elegir el indicador del gráfico de línea en **Por entidad** y **Por sector**. |
| `Parámetro 2` | Departamento, Municipio, Tipo de entidad, Sin filtros | Elegir cómo desagregar el gráfico de línea en **Por sector**. |
| `Indicadores gráfico de líneas` | Saldo en mora, Saldo en riesgo, Saldo total, Tasa mora, Tasa riesgo, Riesgo/Mora (versión sin sufijo) | Existe en el modelo, pero ningún visual lo usa. |

## 3. Relaciones
Todas son muchos a uno con filtro en una dirección (de la tabla "uno" a la tabla "muchos"), sin propiedades especiales en los archivos.

| Desde (muchos) | Hacia (uno) |
|---|---|
| `tb_Balance[CODIGO ENTIDAD]` | `dm_Entidades[CODIGO ENTIDAD]` |
| `tb_Balance[CODRENGLON]` | `dm_Cuentas[CODRENGLON]` |
| `tb_Balance[Fecha reporte]` | `Calendario[Fecha]` |
| `dm_EntidadSeleccionada[CODIGO ENTIDAD]` | `dm_Entidades[CODIGO ENTIDAD]` |

Consecuencia clave **[Inferido de la dirección del filtro]**: un segmentador sobre `dm_Entidades` filtra a `dm_EntidadSeleccionada` y a `tb_Balance`; un segmentador sobre `dm_EntidadSeleccionada` **no** filtra a `dm_Entidades` ni a `tb_Balance`. Por eso las medidas "con entidad seleccionada" y "sin filtros" usan `TREATAS` o `SELECTEDVALUE` para aplicar la entidad elegida a `tb_Balance`.

## 4. Catálogo de cuentas (`dm_Cuentas`)

Cómo se interpreta:
- **Riesgo (14xx):** saldo de cartera por categoría de calificación. La categoría A es riesgo normal (no cuenta como "en riesgo"); B a E sí.
- **Mora (84xx):** saldo de capital por rango de días de mora. Cuenta como mora la cartera con más de 30 días (más de 60 en Vivienda).
- `Cuenta para el indicador` = `Estado = VERDADERO`.

| Cuenta | Tipo | Línea | Altura | Cuenta para el indicador | Descripción |
|---|---|---|---|---|---|
| 140405 | Riesgo | Vivienda | A | No | Categoría a riesgo normal |
| 140410 | Riesgo | Vivienda | B | Sí | Categoría b riesgo aceptable |
| 140415 | Riesgo | Vivienda | C | Sí | Categoría c riesgo apreciable |
| 140420 | Riesgo | Vivienda | D | Sí | Categoría d riesgo significativo |
| 140425 | Riesgo | Vivienda | E | Sí | Categoría e riesgo de incobrabilidad |
| 140505 | Riesgo | Vivienda | A | No | Categoría a riesgo normal |
| 140510 | Riesgo | Vivienda | B | Sí | Categoría b riesgo aceptable |
| 140515 | Riesgo | Vivienda | C | Sí | Categoría c riesgo apreciable |
| 140520 | Riesgo | Vivienda | D | Sí | Categoría d riesgo significativo |
| 140525 | Riesgo | Vivienda | E | Sí | Categoría e riesgo de incobrabilidad |
| 144105 | Riesgo | Consumo con libranza | A | No | Categoría a riesgo normal |
| 144110 | Riesgo | Consumo con libranza | B | Sí | Categoría b riesgo aceptable |
| 144115 | Riesgo | Consumo con libranza | C | Sí | Categoría c riesgo apreciable |
| 144120 | Riesgo | Consumo con libranza | D | Sí | Categoría d riesgo significativo |
| 144125 | Riesgo | Consumo con libranza | E | Sí | Categoría e riesgo de incobrabilidad |
| 144205 | Riesgo | Consumo sin libranza | A | No | Categoría a riesgo normal |
| 144210 | Riesgo | Consumo sin libranza | B | Sí | Categoría b riesgo aceptable |
| 144215 | Riesgo | Consumo sin libranza | C | Sí | Categoría c riesgo apreciable |
| 144220 | Riesgo | Consumo sin libranza | D | Sí | Categoría d riesgo significativo |
| 144225 | Riesgo | Consumo sin libranza | E | Sí | Categoría e riesgo de incobrabilidad |
| 144805 | Riesgo | Microcrédito | A | No | Categoría a riesgo normal |
| 144810 | Riesgo | Microcrédito | B | Sí | Categoría b riesgo aceptable |
| 144815 | Riesgo | Microcrédito | C | Sí | Categoría c riesgo apreciable |
| 144820 | Riesgo | Microcrédito | D | Sí | Categoría d riesgo significativo |
| 144825 | Riesgo | Microcrédito | E | Sí | Categoría e riesgo de incobrabilidad |
| 145505 | Riesgo | Microcrédito | A | No | Categoría a riesgo normal |
| 145510 | Riesgo | Microcrédito | B | Sí | Categoría b riesgo aceptable |
| 145515 | Riesgo | Microcrédito | C | Sí | Categoría c riesgo apreciable |
| 145520 | Riesgo | Microcrédito | D | Sí | Categoría d riesgo significativo |
| 145525 | Riesgo | Microcrédito | E | Sí | Categoría e riesgo de incobrabilidad |
| 146205 | Riesgo | Comercial | A | No | Categoría a riesgo normal |
| 146210 | Riesgo | Comercial | B | Sí | Categoría b riesgo aceptable |
| 146215 | Riesgo | Comercial | C | Sí | Categoría c riesgo apreciable |
| 146220 | Riesgo | Comercial | D | Sí | Categoría d riesgo significativo |
| 146225 | Riesgo | Comercial | E | Sí | Categoría e riesgo de incobrabilidad |
| 146905 | Riesgo | Vivienda | A | No | Categoría a riesgo normal, vivienda |
| 146910 | Riesgo | Vivienda | B | Sí | Categoría b riesgo aceptable, vivienda |
| 146915 | Riesgo | Vivienda | C | Sí | Categoría c riesgo apreciable, vivienda |
| 146920 | Riesgo | Vivienda | D | Sí | Categoría d riesgo significativo, vivienda |
| 146925 | Riesgo | Vivienda | E | Sí | Categoría e riesgo de incobrabilidad, vivienda |
| 146930 | Riesgo | Consumo con libranza | A | No | Categoría a riesgo normal, consumo |
| 146935 | Riesgo | Consumo con libranza | B | Sí | Categoría b riesgo aceptable, consumo |
| 146940 | Riesgo | Consumo con libranza | C | Sí | Categoría c riesgo apreciable, consumo |
| 146945 | Riesgo | Consumo con libranza | D | Sí | Categoría d riesgo significativo, consumo |
| 146950 | Riesgo | Consumo con libranza | E | Sí | Categoría e riesgo de incobrabilidad, consumo |
| 147605 | Riesgo | Microcrédito | A | No | Categoría a riesgo normal |
| 147610 | Riesgo | Microcrédito | B | Sí | Categoría b riesgo aceptable |
| 147615 | Riesgo | Microcrédito | C | Sí | Categoría c riesgo apreciable |
| 147620 | Riesgo | Microcrédito | D | Sí | Categoría d riesgo significativo |
| 147625 | Riesgo | Microcrédito | E | Sí | Categoría e riesgo de incobrabilidad |
| 840505 | Mora | Vivienda | Hasta 60 | No | Capital vigente y hasta 60 días de mora |
| 840510 | Mora | Vivienda | De 61 a 150 | Sí | Capital con mora mayor a 60 días y hasta 150 días |
| 840515 | Mora | Vivienda | De 151 a 360 | Sí | Capital con mora mayor a 150 días y hasta 360 días |
| 840520 | Mora | Vivienda | De 361 a 540 | Sí | Capital con mora mayor a 360 días y hasta 540 días |
| 840525 | Mora | Vivienda | De 541 en adelante | Sí | Capital con mora mayor a 540 días |
| 841005 | Mora | Consumo con libranza | Hasta 30 | No | Capital vigente y hasta 30 días de mora |
| 841010 | Mora | Consumo con libranza | De 31 a 60 | Sí | Capital con mora mayor a 30 días y hasta 60 días |
| 841015 | Mora | Consumo con libranza | De 61 a 90 | Sí | Capital con mora mayor a 60 días y hasta 90 días |
| 841020 | Mora | Consumo con libranza | De 91 a 180 | Sí | Capital con mora mayor a 90 días y hasta 180 días |
| 841025 | Mora | Consumo con libranza | De 181 en adelante | Sí | Capital con mora mayor a 180 días |
| 841505 | Mora | Consumo sin libranza | Hasta 30 | No | Capital vigente y hasta 30 días de mora |
| 841510 | Mora | Consumo sin libranza | De 31 a 60 | Sí | Capital con mora mayor a 30 días y hasta 60 días |
| 841515 | Mora | Consumo sin libranza | De 61 a 90 | Sí | Capital con mora mayor a 60 días y hasta 90 días |
| 841520 | Mora | Consumo sin libranza | De 91 a 180 | Sí | Capital con mora mayor a 90 días y hasta 180 días |
| 841525 | Mora | Consumo sin libranza | De 181 en adelante | Sí | Capital con mora mayor a 180 días |
| 842005 | Mora | Microcrédito | Hasta 30 | No | Capital vigente y hasta 30 días de mora |
| 842010 | Mora | Microcrédito | De 31 a 60 | Sí | Capital con mora mayor a 30 días y hasta 60 días |
| 842015 | Mora | Microcrédito | De 61 a 90 | Sí | Capital con mora mayor a 60 días y hasta 90 días |
| 842020 | Mora | Microcrédito | De 91 a 120 | Sí | Capital con mora mayor a 90 días y hasta 120 días |
| 842025 | Mora | Microcrédito | De 121 en adelante | Sí | Capital con mora mayor a 120 días |
| 842505 | Mora | Comercial | Hasta 30 | No | Capital vigente y hasta 30 días de mora |
| 842510 | Mora | Comercial | De 31 a 90 | Sí | Capital con mora mayor a 30 días y hasta 90 días |
| 842515 | Mora | Comercial | De 91 a 120 | Sí | Capital con mora mayor a 90 días y hasta 120 días |
| 842520 | Mora | Comercial | De 121 a 150 | Sí | Capital con mora mayor a 120 días y hasta 150 días |
| 842525 | Mora | Comercial | De 151 en adelante | Sí | Capital con mora mayor a 150 días |

Grupos de cuentas por línea (saldos de diciembre de 2025, en miles de millones de pesos):
- Vivienda: 1404 (285), 1405 (602), 1469 (10).
- Consumo con libranza: 1441 (6.783), 1469 (24).
- Consumo sin libranza: 1442 (7.531).
- Microcrédito: 1448 (0), 1455 (177), 1476 (1.765).
- Comercial: 1462 (2.304).
- Mora: 8405 Vivienda, 8410 Consumo con libranza, 8415 Consumo sin libranza, 8420 Microcrédito, 8425 Comercial.
- **[Inferido]** 1469 sería cartera de empleados (pendiente de confirmar con el PUC).

## 5. Rangos de mora (`Altura` en cuentas de Mora)

| Línea | Rangos (el primero no cuenta como mora) |
|---|---|
| Vivienda | Hasta 60 / 61 a 150 / 151 a 360 / 361 a 540 / 541 o más |
| Consumo con libranza | Hasta 30 / 31 a 60 / 61 a 90 / 91 a 180 / 181 o más |
| Consumo sin libranza | Hasta 30 / 31 a 60 / 61 a 90 / 91 a 180 / 181 o más |
| Microcrédito | Hasta 30 / 31 a 60 / 61 a 90 / 91 a 120 / 121 o más |
| Comercial | Hasta 30 / 31 a 90 / 91 a 120 / 121 a 150 / 151 o más |

Los rangos cambian de nombre según la línea. `dm_CuentasIndice` agrega el prefijo numérico para ordenar y reemplaza "en adelante" por "o más". Además hay un reemplazo manual de `"4. De 91 a 120"` por `"3. De 91 a 120"` para que "De 91 a 120" quede en la misma posición en Microcrédito (4.º rango) y Comercial (3.º rango). Es un parche frágil.

## 6. Medidas (tabla `Medidas`)

Notación: **Estado** = `Estado = "VERDADERO"` y `Línea` no vacía. Los números de página son las páginas del informe donde se usa cada una.

### Base
| Medida | Lógica | Formato | Dónde se usa |
|---|---|---|---|
| `Saldo` | `SUM(tb_Balance[VALOR EN PESOS])` | Moneda | No se usa |
| `Saldo en mora` | Suma de `VALOR EN PESOS` con `Tipo = "Mora"` y Estado | `0.00` (sin `$`) | Por entidades (tarjeta) |
| `Saldo en riesgo` | Suma con `Tipo = "Riesgo"` y Estado (categorías B a E) | Moneda | Por entidades (tarjeta) |
| `Saldo total` | Suma de las cuentas `Tipo = "Riesgo"` sin filtrar `dm_Cuentas` (categorías A a E) | Moneda | Por entidades (tarjeta) |
| `Tasa mora` | `DIVIDE([Saldo en mora], [Saldo total], 0)` | `0.00 %` | Por entidades (tarjeta y barras), Por sector (barras) |
| `Tasa riesgo` | `DIVIDE([Saldo en riesgo], [Saldo total], 0)` | `0.00 %` | Por entidades (tarjeta y barras) |
| `Riesgo/Mora` | `DIVIDE([Saldo en riesgo], [Saldo en mora], BLANK())` | General | Por entidades (tarjeta y barras) |
| `Ranking Mora` | `RANKX(ALL(dm_Entidades[SIGLA]), [Tasa mora])` | Entero | No se usa |

### Entidad seleccionada, familia "con entidad seleccionada"
Mismas fórmulas base, con `TREATAS(VALUES(dm_EntidadSeleccionada[CODIGO ENTIDAD]), tb_Balance[CODIGO ENTIDAD])`.

| Medida | Dónde se usa |
|---|---|
| `Saldo en mora con entidad seleccionada` | Por entidad (torta), Por sector (torta), Tooltip-Mora |
| `Saldo en riesgo con entidad seleccionada` | Tooltip-Riesgo |
| `Saldo total con entidad seleccionada` | Tooltip-Mora y Tooltip-Riesgo (tarjeta) |
| `Tasa mora con entidad seleccionada` (= mora / total) | Por entidad (línea y barras), Por sector (línea), Tooltip-Mora |
| `Tasa riesgo con entidad seleccionada` (= riesgo / total) | Por entidad y Por sector (barras), Tooltip-Riesgo |
| `Riesgo/Mora con entidad seleccionada` | Por sector (línea) |

### Entidad seleccionada, familia "sin filtros"
Fijan la entidad con `dm_Entidades[SIGLA] = SELECTEDVALUE(dm_EntidadSeleccionada[SIGLA])` y quitan los demás filtros de `SIGLA`. Sirven para comparar la entidad con el contexto filtrado.

| Medida | Lógica | Dónde se usa |
|---|---|---|
| `Tasa mora sin filtros` | `CALCULATE([Tasa mora], ALL(dm_Entidades[SIGLA]), SIGLA = entidad seleccionada)` | Por entidad (barras por línea) |
| `Tasa riesgo sin filtros` | Igual con `[Tasa riesgo]` | Por entidad (barras por línea) |
| `Riesgo/Mora sin filtros` | `CALCULATE(DIVIDE([Saldo en riesgo], [Saldo en mora], BLANK()))` (no fija la entidad) | Por entidad y Por sector (barras por línea) |
| `Saldo en riesgo sin filtros` | Fija la entidad con `SELECTEDVALUE` | No se usa |
| `Saldo en mora sin filtros` | Fórmula igual a `Saldo en mora` (no fija la entidad) | No se usa |

### Apoyo visual
| Medida | Lógica | Dónde se usa |
|---|---|---|
| `Color barra seleccionada` | `#FF8C42` (naranja) si `SELECTEDVALUE(dm_Entidades[SIGLA]) = SELECTEDVALUE(dm_EntidadSeleccionada[SIGLA])`, si no `#1E88E5` (azul) | Por entidades: color de las 3 barras |
| `Nombre de usuario` | Parte de `USERNAME()` anterior a `\` con la primera letra en mayúscula | No se usa |
| `Saludo de bienvenida` | "Buenos días/tardes/noches" según `HOUR(NOW())` + nombre de usuario | No se usa |

Seis medidas no están en ningún visual: `Saldo`, `Ranking Mora`, `Saldo en mora sin filtros`, `Saldo en riesgo sin filtros`, `Saludo de bienvenida` y `Nombre de usuario`. Dos de ellas, las de usuario, dependen de `USERNAME()`, que no sirve en un informe público (nadie inicia sesión).

## 7. Páginas y visuales

Todas miden 1280×720 (los tooltips son más chicos). Navegación: **Inicio** → Por entidades / Por entidad / Por sector, y cada página tiene un botón para volver a Inicio. Cuatro marcadores controlan el panel de ayuda (`InfoPanelEntidades`, `InfoPanelEntidadesOcultar`, `InfoPanelEntidad`, `InfoPanelEntidadOcultar`).

| Página | Contenido | Segmentadores |
|---|---|---|
| **Inicio** | Portada con 3 botones de navegación | Mes (`Calendario[Nombre Mes]`), entidades (`dm_Entidades[SIGLA]`), entidad seleccionada (`dm_EntidadSeleccionada[SIGLA]`) |
| **Por entidades** (benchmarking) | Tarjeta con 6 indicadores; barras de `Tasa mora`, `Tasa riesgo` y `Riesgo/Mora` por entidad, con la entidad seleccionada resaltada | Mes, entidades, entidad seleccionada, `Línea` |
| **Por entidad** | Línea de tiempo con indicador elegible; barras por línea de `Tasa mora`, `Tasa riesgo` y `Riesgo/Mora`; torta de composición de la mora | Mes, rango de fechas, entidad seleccionada, indicador (`Parámetro`) |
| **Por sector** | Igual que Por entidad, pero con desagregación elegible (departamento, municipio, tipo de entidad o sin filtros) | Mes, rango de fechas, tipo de entidad, departamento, municipio, indicador, dimensión |
| **Tooltip-Mora** (oculta) | Saldo total y tabla de saldo y tasa de mora por rango de días (`Altura`) | |
| **Tooltip-Riesgo** (oculta) | Saldo total y tabla de saldo y tasa de riesgo por categoría | |

El fondo de las páginas son imágenes estáticas (6 PNG en `StaticResources/RegisteredResources`, incluido el logo MDA). El tema base es `CY24SU10`, sin paleta propia.

Filtro a nivel de informe: se excluyen las filas con `dm_Entidades[SIGLA]` nula (bloqueado para el usuario).

## 8. Calidad de datos conocida (caché hasta diciembre de 2025)

- **Septiembre 2025:** solo 11 entidades (172 en los otros meses), 10.216 filas en grupos duplicados y negativos en la entidad `2773`. Las tasas de ese mes no son válidas.
- **Enero 2025:** la mora (84xx) del sector es la mitad de la de febrero, así que su tasa de mora (3,89 %) no es comparable con las demás (alrededor de 6,2 % a 6,6 %). Coincide con la fecha desde la que hay datos de mora (confirmado por el dueño), lo que sugiere que enero llegó incompleto.
- **Abril (167), enero y noviembre (171), diciembre (170):** menos entidades que las 172 típicas.
- **Diciembre:** 3.409 cuentas distintas (2.180 en los otros meses); no afecta al catálogo.
- **Conciliación:** las cuentas de mora (84xx) no suman lo mismo que las de cartera (14xx) en Vivienda, Comercial y Consumo sin libranza (diciembre de 2025).
- **Pendiente:** el modelo no sabe qué entidades se fusionan, cambian de nombre o se liquidan (confirmado por el dueño).
