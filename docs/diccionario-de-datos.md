# Diccionario de datos y lógica: MorosidadSectorSolidario

Referencia técnica del modelo semántico y del informe, escrita para consulta futura (por el dueño y por Claude).
Cubre tablas, columnas, relaciones, catálogo de cuentas, medidas (con su lógica y dónde se usan), parámetros de campos y páginas.
Complementa a `docs/conocimiento-informe.md`, que guarda el contexto de negocio, las decisiones y los pendientes.

Última actualización: 2026-11-04. Fuente: archivos del proyecto PBIP, datos del `.pbix` entregado el 2026-11-04 (septiembre de 2025 a agosto de 2026), Circular Básica Contable y Financiera, Título V, y su anexo técnico de indicadores.

---

## 1. Flujo de datos

1. **Fuente pública:** la Superintendencia de la Economía Solidaria publica la información financiera de las entidades vigiladas en https://www.supersolidaria.gov.co/es/content/entidades-vigiladas-que-reportan-informacion.
2. **Dataflows de Power BI Service:** los datos se cargan y limpian en dataflows y de ahí pasan al modelo semántico. El código de esos dataflows no está en este repo.
3. **Modelo semántico** (`MorosidadSectorSolidario.SemanticModel`, cultura `es-CO`, modo Import).
4. **Informe** (`MorosidadSectorSolidario.Report`): 5 páginas visibles y 2 tooltips ocultos, publicado en Power BI Service y embebido en un iframe público (sin inicio de sesión).

Alcance (confirmado): solo cooperativas de ahorro y crédito, y solo desde el corte de **octubre de 2025** (decisión del 2026-11-04; septiembre de 2025 llegó con 11 entidades).

| Tabla del modelo | Origen |
|---|---|
| `tb_Balance` | Dataflow `210af2db-4205-47ea-b3cf-a4b57747d9f8` (workspace `ca53e802-ef7f-4429-b643-fe316dbbff4b`), entidad `tb_Balance` |
| `dm_Entidades` | Dataflow `cc8c038e-23dd-4a57-97cd-4bfe5e79b796` (mismo workspace), entidad `Entidades_PUC` |
| `Calendario` | Dataflow `3103eda1-7a05-4d3d-8016-d3dcdd60938c` (workspace `7d7a12b8-9d3c-48ad-ab9b-018ceb413f07`), entidad `Calendario` |
| `dm_Cuentas` | Tabla escrita en Power Query (`#table`) dentro del modelo. Copia legible en `docs/referencia/catalogo-cuentas-modelo.csv` |
| `dm_EntidadSeleccionada` | Copia de `dm_Entidades` (referencia de la consulta) |
| `Medidas` | Tabla vacía que solo contiene las medidas |
| `Parámetro`, `Parámetro 2`, `Indicadores gráfico de líneas` | Parámetros de campos (tablas calculadas) |

## 2. Tablas y columnas

### `tb_Balance` (hechos)
Una fila por fecha, entidad y cuenta del PUC (todas las clases, 1 a 9, a 6 dígitos; incluye las filas de total de cada cuenta, terminadas en `00`).

| Columna | Tipo | Descripción |
|---|---|---|
| `Fecha reporte` | Fecha | Cierre de mes del reporte. Se filtra `>= 2025-10-01`. |
| `CODIGO ENTIDAD` | Texto | Código de la entidad en la Supersolidaria. Une con `dm_Entidades`. |
| `CODRENGLON` | Texto | Código de cuenta del PUC. Une con `dm_Cuentas`. |
| `VALOR EN PESOS` | Entero | Saldo de la cuenta en pesos. Las cuentas de deterioro (CR) vienen en positivo. |

Lógica de carga: fechas desde 2025-10-01, cruce con `dm_Entidades` y descarte de las filas cuya entidad no está allí.

### `dm_Entidades` (dimensión)
174 entidades: 140 especializadas, 32 multiactivas y 2 integrales, todas con ahorro y crédito. `CODIGO ENTIDAD` y `SIGLA` son únicos.

| Columna | Tipo | Descripción |
|---|---|---|
| `CODIGO ENTIDAD` | Texto | Llave de la entidad. |
| `TIPO ENTIDAD SOLIDARIA` | Texto | Tipo de entidad. |
| `MUNICIPIO`, `DEPARTAMENTO` | Texto | Domicilio. |
| `SIGLA` | Texto | Sigla; si viene vacía se usa el nombre completo. |
| `Filtros todos` | Texto (calculada) | Constante `"Sin filtros"`, opción "sin dimensión" de `Parámetro 2`. |

### `dm_EntidadSeleccionada`
Copia de `dm_Entidades`. Se usa para elegir **la** entidad a analizar sin filtrar al resto. La elige siempre el usuario (no hay valor por defecto).

### `dm_Cuentas` (catálogo de cuentas)
152 cuentas. Catálogo completo en `docs/referencia/catalogo-cuentas-modelo.csv`; resumen en la sección 4.

| Columna | Tipo | Descripción |
|---|---|---|
| `CODRENGLON` | Texto | Código de cuenta (6 dígitos). Se ordena por `Índice`. |
| `Descripción` | Texto | Grupo de cuenta y categoría o franja. |
| `Tipo` | Texto | `Riesgo` (14xx por calificación), `Mora` (84xx por altura de mora), `Deterioro` (deterioro individual de capital) o `Deterioro general` (1468). |
| `Línea` | Texto | Vivienda, Consumo con libranza, Consumo sin libranza, Microcrédito, Comercial. **Vacía** en el deterioro de consumo (1445 y empleados consumo), que no separa con y sin libranza, y en el deterioro general. |
| `Modalidad` | Texto | Vivienda, Consumo, Microcrédito, Comercial (ordenada por `Orden modalidad`). Agrupa las dos líneas de consumo. Es el nivel donde se calcula la cobertura. Vacía en el deterioro general. |
| `Producto` | Texto | Detalle: Vivienda con / sin libranza, Consumo con / sin libranza, Empleados vivienda / consumo, Microcrédito inmobiliario / empresarial, **Productivo**, Comercial, Microcrédito y productivo (8420), Deterioro general. |
| `Altura` | Texto | Riesgo y Deterioro: categoría A a E. Mora: franja numerada (`1. Hasta 30` … `5. De 121 o más`). |
| `Estado` | Texto | Riesgo y Mora: `VERDADERO` si cuenta para el indicador (categorías B a E; franjas sobre el umbral de mora), `FALSO` si no. Vacío en deterioro. |
| `Índice` | Entero | Orden del catálogo. |
| `Orden modalidad` | Entero (oculta) | 1 Vivienda, 2 Consumo, 3 Microcrédito, 4 Comercial. |

### `Calendario`
Fechas diarias desde 2025-10-01 (hasta 2030). Columnas `Fecha`, `Año` (texto), `Mes`, `Nombre Mes` (ordenado por `Mes`). Tiene una jerarquía de fechas automática (`LocalDateTable_…`).

### Parámetros de campos
| Tabla | Opciones | Uso |
|---|---|---|
| `Parámetro` | Riesgo/Mora, Saldo en mora, Saldo en riesgo, Saldo total, Tasa mora, Tasa riesgo, **Cobertura** (todas "con entidad seleccionada") | Indicador del gráfico de línea en **Por entidad** y **Por sector**. |
| `Parámetro 2` | Departamento, Municipio, Tipo de entidad, Sin filtros | Desagregación del gráfico de línea en **Por sector**. |
| `Indicadores gráfico de líneas` | Las mismas opciones de `Parámetro`, en versión sin sufijo | Ningún visual lo usa. |

## 3. Relaciones
Todas muchos a uno, filtro en una dirección.

| Desde (muchos) | Hacia (uno) |
|---|---|
| `tb_Balance[CODIGO ENTIDAD]` | `dm_Entidades[CODIGO ENTIDAD]` |
| `tb_Balance[CODRENGLON]` | `dm_Cuentas[CODRENGLON]` |
| `tb_Balance[Fecha reporte]` | `Calendario[Fecha]` |
| `dm_EntidadSeleccionada[CODIGO ENTIDAD]` | `dm_Entidades[CODIGO ENTIDAD]` |

Un segmentador sobre `dm_EntidadSeleccionada` no filtra `tb_Balance`; por eso la familia "con entidad seleccionada" usa `TREATAS`.

## 4. Catálogo de cuentas (`dm_Cuentas`)

Definiciones según el Título V (Circular Básica Contable y Financiera) y su anexo técnico.

### Cartera por calificación (Tipo `Riesgo`, subcuentas 05 A, 10 B, 15 C, 20 D, 25 E)
| Grupo | Nombre | Línea | Modalidad | Producto |
|---|---|---|---|---|
| 1404 | Vivienda con libranza | Vivienda | Vivienda | Vivienda con libranza |
| 1405 | Vivienda sin libranza | Vivienda | Vivienda | Vivienda sin libranza |
| 1469 (05 a 25) | Empleados, vivienda | Vivienda | Vivienda | Empleados vivienda |
| 1411, 1441 | Consumo con libranza | Consumo con libranza | Consumo | Consumo con libranza |
| 1469 (30 a 50) | Empleados, consumo | Consumo con libranza | Consumo | Empleados consumo |
| 1412, 1442 | Consumo sin libranza | Consumo sin libranza | Consumo | Consumo sin libranza |
| 1448 | Microcrédito inmobiliario | Microcrédito | Microcrédito | Microcrédito inmobiliario |
| 1454, 1455 | Microcrédito empresarial | Microcrédito | Microcrédito | Microcrédito empresarial |
| 1476 | Créditos productivos | Microcrédito | Microcrédito | Productivo |
| 1461, 1462 | Comercial | Comercial | Comercial | Comercial |

1411, 1412, 1454 y 1461 son cuentas del anexo técnico que no están en el catálogo de 2023; hoy están en $0, se incluyen para cuando tengan saldo.

**Productivos (1476):** la cartera productiva reemplaza al microcrédito (confirmado por el dueño), así que va en la línea y modalidad Microcrédito. En la cuenta 84 no tiene renglón propio: se reporta dentro de 8420 (validado: en 38 de 43 entidades 8420 = 1448 + 1455 + 1476 al peso).

### Cartera por altura de mora (Tipo `Mora`, cuenta 84)
| Grupo | Línea | Modalidad | Franjas (la primera no es mora) |
|---|---|---|---|
| 8405 | Vivienda | Vivienda | 1. Hasta 60 / 2. De 61 a 150 / 3. De 151 a 360 / 4. De 361 a 540 / 5. De 541 o más |
| 8410 | Consumo con libranza | Consumo | 1. Hasta 30 / 2. De 31 a 60 / 3. De 61 a 90 / 4. De 91 a 180 / 5. De 181 o más |
| 8415 | Consumo sin libranza | Consumo | igual que 8410 |
| 8420 | Microcrédito (incluye productivos) | Microcrédito | 1. Hasta 30 / 2. De 31 a 60 / 3. De 61 a 90 / 4. De 91 a 120 / 5. De 121 o más |
| 8425 | Comercial | Comercial | 1. Hasta 30 / 2. De 31 a 90 / 3. De 91 a 120 / 4. De 121 a 150 / 5. De 151 o más |

Umbral de mora: más de 30 días, más de 60 en vivienda (anexo técnico). El catálogo de cuentas de 2023 no trae la cuenta 84 (se creó en 2025); su estructura se dedujo de los datos.

### Deterioro (Tipo `Deterioro` y `Deterioro general`)
| Grupo | Nombre | Línea | Modalidad | Subcuentas por categoría |
|---|---|---|---|---|
| 1408 | Deterioro vivienda | Vivienda | Vivienda | 05 A … 25 E |
| 1471 (05 a 25) | Deterioro empleados vivienda | Vivienda | Vivienda | 05 A … 25 E |
| 1445 | Deterioro consumo | (vacía) | Consumo | 10 A, 20 B, 30 C, 40 D, 50 E (y 05, 15, 25, 35, 45 del anexo, hoy en $0) |
| 1471 (30 a 50) | Deterioro empleados consumo | (vacía) | Consumo | 30 A … 50 E |
| 1451 | Deterioro microcrédito inmobiliario | Microcrédito | Microcrédito | 05 A … 25 E |
| 1458 | Deterioro microcrédito empresarial | Microcrédito | Microcrédito | como 1445 |
| 1479 | Deterioro créditos productivos | Microcrédito | Microcrédito | 05 A … 25 E |
| 1465 | Deterioro comercial | Comercial | Comercial | como 1445 |
| 146805, 146810 | Deterioro general y general adicional | (vacía) | (vacía) | — |

El deterioro se toma por subcuenta (como en el anexo técnico) para poder calcular la cobertura C, D y E. En agosto de 2026 la suma de subcuentas difiere del total de la cuenta en menos de 0,1 %.

## 5. Medidas (tabla `Medidas`)

Patrón de los denominadores: `ALLEXCEPT(dm_Cuentas, dm_Cuentas[Línea], dm_Cuentas[Modalidad], dm_Cuentas[Producto])` quita los filtros de `Tipo`, `Estado` y `Altura`, y conserva la línea, la modalidad y el producto del contexto. Así cada tasa se divide por la cartera **de su misma línea**.

### Saldos
| Medida | Lógica | Formato |
|---|---|---|
| `Saldo en riesgo` | Cuenta 14, categorías B a E (`Tipo = Riesgo`, `Estado = VERDADERO`) | Moneda |
| `Saldo en mora` | Cuenta 84, franjas sobre el umbral (`Tipo = Mora`, `Estado = VERDADERO`) | Moneda |
| `Saldo total` | Cartera bruta: cuenta 14, categorías A a E, de la línea del contexto | Moneda |
| `Cartera total cuenta 84` | Cuenta 84, todas las franjas, de la línea del contexto | Moneda |
| `Saldo` | `SUM(VALOR EN PESOS)` sin filtros de cuenta (sin uso) | Moneda |

### Indicadores de calidad
| Medida | Lógica | Referencia |
|---|---|---|
| `Tasa riesgo` | `Saldo en riesgo / Saldo total` | Indicador de calidad por riesgo |
| `Tasa mora` | `Saldo en mora / Cartera total cuenta 84`. En blanco si la entidad no reportó la cuenta 84 | Indicador de cartera por mora (el denominador sale de la misma cuenta 84, decisión del dueño) |
| `Riesgo/Mora` | `Tasa riesgo / Tasa mora` | Relación de cartera por riesgo y por mora |

En el total de una entidad o del sector, `Saldo total` y `Cartera total cuenta 84` son prácticamente iguales (agosto de 2026: 20.954,4 mil millones en ambas). Por línea pueden diferir cuando una entidad clasifica distinto en la 14 y en la 84 (sección 7).

### Deterioro y cobertura
| Medida | Lógica |
|---|---|
| `Deterioro individual` | Suma del deterioro individual de capital de la modalidad del contexto |
| `Deterioro general` | 1468. Queda en blanco cuando hay filtro de línea, modalidad o producto |
| `Deterioro total` | Individual + general |
| `Cobertura cartera en riesgo` | `Deterioro total / Saldo en riesgo`. **Por modalidad** solo entra el deterioro individual; **en el total** se suma el general (decisión del dueño) |
| `Deterioro sobre cartera bruta` | `Deterioro total / Saldo total` (fórmula literal del anexo para la "cobertura de la cartera total en riesgo"; indicador secundario) |
| `Cobertura individual CDE` | Deterioro individual de categorías C, D y E / cartera en C, D y E |

La cobertura se calcula por **modalidad**: en consumo el deterioro no separa con y sin libranza, así que por `Línea` la cobertura de consumo queda en blanco.

### Familia "con entidad seleccionada"
`CALCULATE([medida base], TREATAS(VALUES(dm_EntidadSeleccionada[CODIGO ENTIDAD]), tb_Balance[CODIGO ENTIDAD]))` para: Saldo en mora, Saldo en riesgo, Saldo total, Cartera total cuenta 84, Tasa mora, Tasa riesgo, Riesgo/Mora, Deterioro individual, Deterioro general, Deterioro total, Cobertura cartera en riesgo, Deterioro sobre cartera bruta y Cobertura individual CDE. Sin entidad elegida, equivale al conjunto de entidades visibles (el sector filtrado).

### Familia "sin filtros"
| Medida | Lógica | Dónde se usa |
|---|---|---|
| `Tasa mora sin filtros`, `Tasa riesgo sin filtros` | Fijan la entidad con `SELECTEDVALUE(dm_EntidadSeleccionada[SIGLA])` | Filtros de las barras por línea de Por entidad |
| `Riesgo/Mora sin filtros` | Igual a `Riesgo/Mora` (no quita filtros; se conserva el nombre porque lo usan visuales) | Barras de Por entidad y Por sector |
| `Saldo en riesgo sin filtros` | Fija la entidad | Sin uso |
| `Saldo en mora sin filtros` | Igual a `Saldo en mora` | Sin uso |

### Otras
| Medida | Uso |
|---|---|
| `Ranking Mora` | Sin uso |
| `Cantidad de entidades` | Control de cobertura de datos |
| `Color barra seleccionada` | Resalta la entidad seleccionada en Por entidades |
| `Nombre de usuario`, `Saludo de bienvenida` | Sin uso (el informe es público, no hay usuario autenticado) |

## 6. Páginas y visuales

| Página | Contenido |
|---|---|
| **Inicio** | Portada con botones de navegación y segmentadores de mes y entidades |
| **Por entidades** | Tarjeta con 6 indicadores; barras de Tasa mora, Tasa riesgo y Riesgo/Mora por entidad, con la entidad seleccionada resaltada |
| **Por entidad** | Línea de tiempo con indicador elegible (incluye Cobertura); barras por línea; torta de composición de la mora |
| **Por sector** | Igual que Por entidad, con desagregación elegible |
| **Cobertura de cartera** (nueva, 2026-11-04) | Tendencia de la cobertura; tarjetas (cobertura, deterioro / cartera bruta, cobertura C-D-E, deterioro total); barras por modalidad de cobertura, cobertura C-D-E y tasa de riesgo. Usa la familia "con entidad seleccionada": sin entidad elegida muestra el sector |
| **Tooltip-Mora**, **Tooltip-Riesgo** (ocultas) | Saldo y tasa por franja de mora o por categoría |

La página "Cobertura de cartera" no tiene botón en Inicio (las páginas existentes no se modificaron); se llega por la lista de páginas.

## 7. Calidad de datos conocida

- **Septiembre 2025:** 11 entidades. Fuera del alcance desde el 2026-11-04.
- **Clasificación distinta entre la cuenta 14 y la 84** (validado de octubre de 2025 a agosto de 2026): en el 94,5 % de los casos entidad-mes, (1448 + 1454 + 1455 + 1476) = 8420 y (1461 + 1462) = 8425 al peso. Excepciones recurrentes: `3246` (unos $197 mil millones al mes de microcrédito o productivo reportados como comercial en la 84), `1198` (unos $15 mil millones) y `3341` (unos $7 mil millones). `561` no reporta la 8425 en febrero y junio de 2026. Por eso la tasa de mora usa como denominador la propia cuenta 84.
- El diagnóstico de 2026 (hojas del Excel, tipos de dato, directorio estático) está en `docs/diagnostico-datos-2026.md`. Con los datos del 2026-11-04, agosto de 2026 ya tiene 167 entidades.
- **Pendiente:** el modelo no sabe qué entidades se fusionan, cambian de nombre o se liquidan.
