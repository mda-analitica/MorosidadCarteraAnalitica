# Conocimiento del informe: Morosidad del Sector Solidario

Documento vivo. Registra lo que sabemos del informe de Power BI, las decisiones tomadas y lo que falta por aclarar.
Última actualización: 2026-11-03. El detalle técnico (columnas, catálogo de cuentas, lógica de medidas y uso en visuales) está en `docs/diccionario-de-datos.md`.

Convenciones: **[Confirmado]** lo dijo el dueño del informe. **[Verificado]** se comprobó en los archivos del modelo. **[Inferido]** es una deducción pendiente de confirmar.

---

## 1. Propósito, audiencia y distribución

- **[Confirmado] Usuarios:** especialistas y analistas de riesgo, analistas de crédito, gerentes de cooperativas y directivos financieros.
- **[Confirmado] Pregunta de 30 segundos:** cómo está la cartera de mi cooperativa frente al resto de las entidades, y cómo está por sí sola.
- **[Confirmado] Distribución:** Power BI Service, incrustado en un iframe dentro de una URL pública, sin inicio de sesión.
- **[Confirmado] Datos:** públicos, extraídos de la página de la Superintendencia de la Economía Solidaria. Ninguno es confidencial.
- **[Confirmado] Motivo de mejora:** principalmente la parte visual (estética).
- **Implicación:** con acceso público no hay usuario autenticado, así que las medidas basadas en `USERNAME()` no sirven (ver 1b).

## 1b. Respuestas y decisiones del dueño (2026-11-03)

- **[Confirmado] Documentación:** en archivos `.md`, con un diccionario de datos y las lógicas, escrito para que Claude lo consulte en el futuro (ver `docs/diccionario-de-datos.md`).
- **[Confirmado] Edición:** solo el dueño trabaja en el informe. Claude hará los cambios directamente en los archivos del proyecto.
- **[Confirmado] Alcance de los cambios:** se puede cambiar el diseño, el modelo, las tablas, las medidas y la lógica de UX y UI del informe.
- **[Confirmado] Orden de trabajo:** por ahora **no** se hacen cambios (ni reorganizar medidas ni renombrarlas). Primero hay que entender todo el modelo.
- **[Confirmado] Usuario:** `USERNAME()` solo sirve con usuario autenticado, y el informe es público. Por eso `Saludo de bienvenida` y `Nombre de usuario` no sirven. **[Verificado]** Ningún visual las usa.
- **[Confirmado] Entidad seleccionada:** la elige siempre el usuario; no hay valor por defecto.
- **[Confirmado] Celular:** aún no hay vista móvil. En el futuro se diseñará un visor público para celulares.
- **[Confirmado] Entidades:** el modelo todavía no sabe cuáles se fusionan, cambian de nombre o se liquidan.
- **[Confirmado] Alcance:** el informe por ahora solo mide a las cooperativas de ahorro y crédito. Las 174 entidades de `dm_Entidades` (140 especializadas, 32 multiactivas y 2 integrales, todas con ahorro y crédito) están dentro del alcance.
- **[Confirmado] Fecha de inicio:** desde 2025-01-01, porque desde esa fecha hay información de cartera en mora.
- **[Confirmado] Fuente:** https://www.supersolidaria.gov.co/es/content/entidades-vigiladas-que-reportan-informacion; los datos pasan por flujos de datos (dataflows) y luego al modelo semántico.
- **[Verificado] Uso de medidas (respuesta a "¿cuál se usa en qué visual?"):** las dos familias de entidad seleccionada se usan, y ambas son necesarias hoy. Las medidas sin uso en visuales son `Saldo`, `Ranking Mora`, `Saldo en mora sin filtros`, `Saldo en riesgo sin filtros`, `Saludo de bienvenida` y `Nombre de usuario`. Ver la tabla completa en el diccionario.

## 2. Definiciones de negocio

- **[Confirmado] Riesgo / Mora** (idea de fondo): `Riesgo/Mora` es un indicador de alerta temprana. Si el riesgo crece frente a la mora, es probable que la mora suba en el corto plazo.
- **[Confirmado] Cartera en riesgo:** toda la cartera con calificación distinta de A (categorías B a E).
- **[Confirmado] Tasa de mora:** cartera con mora mayor a 30 o 60 días, según la línea de crédito, sobre la cartera total.
- **[Verificado] Umbral de mora por línea** (catálogo `dm_Cuentas`):

| Línea | Es mora desde |
|---|---|
| Vivienda | más de 60 días |
| Consumo con libranza | más de 30 días |
| Consumo sin libranza | más de 30 días |
| Microcrédito | más de 30 días |
| Comercial | más de 30 días (rango 31 a 90 en un solo tramo) |

- **[Verificado] Riesgo:** `Estado = "VERDADERO"` equivale a categorías B a E. La categoría A (`FALSO`) queda fuera de `Saldo en riesgo`, pero entra en `Saldo total`.
- **Referencia normativa:** los indicadores de supervisión se agrupan en **CAMEL**. El dueño pasará el documento más adelante (ver pendientes).

## 3. Origen de los datos

- **[Verificado]** Todo llega de **dataflows de Power BI Service** (`PowerPlatform.Dataflows`). La carga y limpieza están fuera de este repo.
  - `tb_Balance`: workspace `ca53e802-ef7f-4429-b643-fe316dbbff4b`, dataflow `210af2db-4205-47ea-b3cf-a4b57747d9f8`, entidad `tb_Balance`.
  - `dm_Entidades`: mismo workspace, dataflow `cc8c038e-23dd-4a57-97cd-4bfe5e79b796`, entidad `Entidades_PUC`.
  - `Calendario`: workspace `7d7a12b8-9d3c-48ad-ab9b-018ceb413f07`, dataflow `3103eda1-7a05-4d3d-8016-d3dcdd60938c`, entidad `Calendario`.
- **[Verificado]** `tb_Balance` carga desde 2025-01-01 y descarta las filas de entidades que no están en `dm_Entidades`.
- **[Verificado]** `dm_Entidades` solo incluye entidades de tipo: especializada de ahorro y crédito, integral con ahorro y crédito, multiactiva con ahorro y crédito. La sigla se rellena con el nombre cuando está vacía.
- **[Verificado]** `dm_Cuentas` y `dm_CuentasIndice` son tablas escritas a mano dentro del modelo (datos incrustados), no vienen de una fuente.

## 4. Modelo de datos

Tablas: `tb_Balance` (hechos), `dm_Entidades`, `dm_EntidadSeleccionada`, `dm_Cuentas`, `Calendario`, `Medidas`, tres tablas de parámetros de campos (`Parámetro`, `Parámetro 2`, `Indicadores gráfico de líneas`) y tablas de fechas automáticas.

Relaciones:
- `tb_Balance[CODIGO ENTIDAD]` → `dm_Entidades[CODIGO ENTIDAD]`
- `tb_Balance[CODRENGLON]` → `dm_Cuentas[CODRENGLON]`
- `tb_Balance[Fecha reporte]` → `Calendario[Fecha]`
- `dm_EntidadSeleccionada[CODIGO ENTIDAD]` → `dm_Entidades[CODIGO ENTIDAD]`

`dm_Cuentas` (75 cuentas del PUC):
- `Línea`: línea de crédito (Vivienda, Consumo con libranza, Consumo sin libranza, Microcrédito, Comercial).
- `Altura`: en cuentas de Mora es el rango de días; en cuentas de Riesgo es la categoría A a E.
- `Tipo`: `Riesgo` (cuentas 14xx) o `Mora` (cuentas 84xx).
- `dm_CuentasIndice` numera los rangos de mora para ordenarlos y cambia "en adelante" por "o más". Hay un reemplazo manual `"4. De 91 a 120"` → `"3. De 91 a 120"` para unificar el orden entre Microcrédito y Comercial. Es un parche frágil.

## 4b. Hallazgos al leer los datos del caché (`cache.abf`)

Se abrió el caché local del modelo (formato Analysis Services con compresión XPress9) con la librería `pbixray`. Es una **copia de los datos cargados en Desktop**, no necesariamente la versión publicada. Todos los valores de abajo son **[Verificado]** contra ese caché.

**Volumen y cobertura**
- `tb_Balance`: 4.335.629 filas, 4 columnas, de enero a diciembre de 2025 (12 cierres de mes). 172 entidades con datos (`dm_Entidades` tiene 174; las entidades `1271` y `1376` no tienen balance).
- `Calendario`: 2.190 días (hasta 2028).
- Cada mes trae 2.180 cuentas distintas por entidad, salvo **diciembre, que trae 3.409**. Es decir, aparecieron unas 1.229 cuentas nuevas en diciembre (catálogo del PUC ampliado). No afectan al catálogo de 75 cuentas del informe.

**Problemas de calidad detectados** (afectan las cifras que muestra el informe)
1. **Septiembre 2025 está casi vacío:** solo 11 entidades (172 en los demás meses) y el total de cartera cae de ~19 billones a ~0,96 billones. La tasa de mora de ese mes sale 5,15 % en el sector (los demás meses ~6,2 a 6,5 %), un valor falso.
2. **Septiembre 2025 tiene filas duplicadas:** 10.216 filas pertenecen a 4.222 combinaciones repetidas de fecha, entidad y cuenta (3.834 con valores idénticos, 388 con valores distintos). Todo el duplicado está en septiembre. También hay valores negativos en cuentas de cartera de la entidad `2773`, solo en septiembre.
3. **Enero 2025 tiene la mora incompleta:** la mora del sector es ~50 % menor que en febrero (Comercial 943 frente a 2.058 miles de millones; Consumo con libranza 2.778 frente a 5.739), mientras que la cartera por categoría de riesgo es estable. La tasa de mora de enero sale 3,89 % frente a ~6,6 % en febrero. Es una caída artificial (probablemente las cuentas 84xx no se reportaron completas en enero).
4. **Otros meses con menos entidades:** abril (167), enero (171), noviembre (171) y diciembre (170), frente a 172. Esto ya mueve levemente los totales y el ranking.

**Tasas del sector por mes** (con las definiciones actuales del informe)

| Cierre | Cartera (billones) | Tasa mora | Tasa riesgo | Riesgo/Mora |
|---|---|---|---|---|
| Ene 2025 | 17,9 | 3,89 % (incompleto) | 8,76 % | 2,25 |
| Feb 2025 | 18,1 | 6,59 % | 8,77 % | 1,33 |
| Mar 2025 | 18,3 | 6,28 % | 8,89 % | 1,42 |
| Abr 2025 | 18,2 | 6,39 % | 8,73 % | 1,37 |
| May 2025 | 18,5 | 6,18 % | 8,59 % | 1,39 |
| Jun 2025 | 18,6 | 6,45 % | 9,04 % | 1,40 |
| Jul 2025 | 18,8 | 6,49 % | 8,73 % | 1,34 |
| Ago 2025 | 18,9 | 6,46 % | 8,66 % | 1,34 |
| Sep 2025 | 0,96 (incompleto) | 5,15 % (inválido) | 6,24 % (inválido) | 1,21 |
| Oct 2025 | 19,3 | 6,25 % | 8,33 % | 1,33 |
| Nov 2025 | 19,5 | 6,18 % | 8,31 % | 1,35 |
| Dic 2025 | 19,5 | 5,89 % | 7,97 % | 1,35 |

**Cuentas "repetidas" del catálogo: resuelto (no hay doble conteo)**
Las cuentas con la misma línea en grupos distintos (14xx) son cartera distinta, con saldos independientes a diciembre de 2025, en miles de millones de pesos:
- Vivienda: grupo 1404 (285), 1405 (602) y 1469 (10).
- Consumo con libranza: 1441 (6.783) y 1469 (24).
- Consumo sin libranza: 1442 (7.531).
- Microcrédito: 1448 (0), 1455 (177) y 1476 (1.765).
- Comercial: 1462 (2.304).
- **[Inferido]** El grupo 1469 parece ser cartera de empleados; falta confirmarlo con el PUC oficial.

**Conciliación mora frente a cartera (diciembre 2025)**
La suma de las cuentas de mora (84xx) no coincide con la de cartera (14xx) en algunas líneas: Vivienda 921 frente a 896, Comercial 2.490 frente a 2.304, Consumo sin libranza 7.576 frente a 7.531 (mora mayor que la cartera). En Consumo con libranza y Microcrédito sí se parecen. Esto significa que la `Tasa mora` usa en el numerador cuentas que no se concilian con el denominador. Falta definir cuál es la base correcta (ver pendientes).

## 5. Medidas (24, tabla `Medidas`)

- **Base:** `Saldo`, `Saldo en mora`, `Saldo en riesgo`, `Saldo total` (suma de las cuentas de riesgo, categorías A a E).
- **Indicadores:** `Tasa mora`, `Tasa riesgo`, `Riesgo/Mora`, `Ranking Mora`.
- **Entidad seleccionada, familia "sin filtros":** `Saldo en mora sin filtros`, `Saldo en riesgo sin filtros`, `Tasa mora sin filtros`, `Tasa riesgo sin filtros`, `Riesgo/Mora sin filtros`.
- **Entidad seleccionada, familia "con entidad seleccionada":** `Saldo en mora`, `Saldo en riesgo`, `Saldo total`, `Tasa mora`, `Tasa riesgo`, `Riesgo/Mora` (todas con el sufijo "con entidad seleccionada").
- **Apoyo visual:** `Color barra seleccionada` (naranja `#FF8C42` para la entidad seleccionada, azul `#1E88E5` para las demás), `Saludo de bienvenida`, `Nombre de usuario`.

Observaciones:
1. `Saldo en mora sin filtros` no fija la entidad (es igual a `Saldo en mora`). Los otros "sin filtros" sí.
2. `Nombre de usuario` falla si `USERNAME()` no contiene `\`.
3. Formatos inconsistentes: `Saldo en mora` usa `0.00` sin `$`; `Saldo en riesgo sin filtros` usa `0`.
4. Las dos familias de entidad seleccionada resuelven casi lo mismo y podrían unificarse.
5. El comentario de `Color barra seleccionada` dice "Amarillo", pero el color es naranja.

## 6. Páginas del informe

Tamaño 1280×720. Orden: Inicio, Por entidades, Por entidad, Por sector, Tooltip-Riesgo (oculta), Tooltip-Mora (oculta).

- **Inicio:** segmentadores de mes y entidad, aviso "mantén Ctrl para seleccionar varias entidades" y tres botones: benchmarking de cooperativas (Por entidades), informe por entidad (Por entidad), indicadores por sector (Por sector).
- **Por entidades:** benchmarking con barras de tasa de mora, tasa de riesgo y Riesgo/Mora por entidad, y una tarjeta con los seis indicadores.
- **Por entidad:** una cooperativa. Línea de tiempo con indicador elegible, tasas y saldos por línea de crédito, y torta con la composición de la mora.
- **Por sector:** misma estructura, con filtros por tipo de entidad, departamento y municipio, y dimensión elegible para la línea de tiempo.
- **Tooltips:** muestran saldo total, saldo en mora o riesgo y tasa por rango (`Altura`).
- **Marcadores (4):** abren y cierran un panel de ayuda en Por entidades y Por entidad.

## 7. Estado visual actual

- El diseño depende de **imágenes de fondo estáticas** (6 PNG, incluido el logo MDA) con rectángulos y flechas de ayuda. Si se mueven los visuales, el fondo deja de calzar.
- Tema base: `CY24SU10` (el predeterminado), sin paleta corporativa propia.
- 15 botones, 10 imágenes y 16 segmentadores en total.

## 8. Ideas para después

- **Riesgo/Mora como percentil:** convertir el ratio en un percentil (por ejemplo, contra el resto de entidades o contra su propia historia) para que sea más fácil de leer y definir umbrales de alerta. Diseñar después.
- Unificar las dos familias de medidas de entidad seleccionada.
- Reemplazar o quitar el saludo de bienvenida si no funciona en la versión pública.
- Carpetas de visualización en `Medidas` y formatos consistentes.

## 9. Pendientes

**Recordar al dueño:**
- [ ] **Volver a pasar el documento de CAMEL** para leerlo y citar las definiciones oficiales.

**Por verificar:**
- [ ] Confirmar con el PUC oficial qué es el grupo 1469 (parece cartera de empleados).
- [ ] **En curso (2026-11-03):** el dueño está revisando los dataflows para actualizar el modelo y volver a subirlo; cuando lo suba, releer el caché y revalidar septiembre y enero. Septiembre 2025 (11 entidades, filas duplicadas y negativos) y enero 2025 (mora a la mitad): confirmar en el dataflow si es un error de carga o un dato así en la fuente, y decidir cómo tratarlos (excluir el mes, corregir en el dataflow o avisar en el informe).
- [ ] Conciliación de mora (84xx) frente a cartera (14xx): decidir cuál es la base correcta del denominador de las tasas.
- [ ] El caché termina en diciembre de 2025; confirmar si la versión publicada tiene datos más recientes.
- [ ] Si `Comercial` agrupa 31 a 90 días por norma.

**Preguntas sin responder** (se van planteando de a una):
- Tema visual: conservar la estructura actual y mejorar colores, tipografía y fondos, o rehacer el diseño desde cero (el dueño ya autorizó cambiar el diseño, falta la dirección).
- Guía de marca o paleta; qué es lo que menos gusta hoy; tamaño del iframe.
- Refresco programado y frecuencia de llegada de datos nuevos.
- Qué segmentadores casi no se usan.
- Un número de control del sector para validar el modelo.
- Cómo se deben tratar las fusiones y liquidaciones de entidades cuando se defina.
