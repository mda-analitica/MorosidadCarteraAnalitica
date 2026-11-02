# Conocimiento del informe: Morosidad del Sector Solidario

Documento vivo. Registra lo que sabemos del informe de Power BI, las decisiones tomadas y lo que falta por aclarar.
Última actualización: 2026-11-02.

Convenciones: **[Confirmado]** lo dijo el dueño del informe. **[Verificado]** se comprobó en los archivos del modelo. **[Inferido]** es una deducción pendiente de confirmar.

---

## 1. Propósito, audiencia y distribución

- **[Confirmado] Usuarios:** especialistas y analistas de riesgo, analistas de crédito, gerentes de cooperativas y directivos financieros.
- **[Confirmado] Pregunta de 30 segundos:** cómo está la cartera de mi cooperativa frente al resto de las entidades, y cómo está por sí sola.
- **[Confirmado] Distribución:** Power BI Service, incrustado en un iframe dentro de una URL pública, sin inicio de sesión.
- **[Confirmado] Datos:** públicos, extraídos de la página de la Superintendencia de la Economía Solidaria. Ninguno es confidencial.
- **[Confirmado] Motivo de mejora:** principalmente la parte visual (estética).
- **Implicación:** con acceso público no hay usuario autenticado. Las medidas `Nombre de usuario` y `Saludo de bienvenida` (basadas en `USERNAME()`) probablemente no funcionan en la versión publicada. Pendiente de verificar.

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
- [ ] Cuentas repetidas en `dm_Cuentas` para la misma línea (Microcrédito en 1448, 1455 y 1476; Vivienda en 1405 y 1469). Confirmar si es cartera distinta o doble conteo. Requiere los datos de `tb_Balance` o el PUC oficial.
- [ ] Si el saludo de bienvenida se ve bien en la versión publicada.
- [ ] Si `Comercial` agrupa 31 a 90 días por norma.

**Preguntas sin responder** (se van planteando de a una):
- Tema visual: conservar la estructura actual y mejorar colores, tipografía y fondos, o rehacer el diseño desde cero.
- Guía de marca o paleta; qué es lo que menos gusta hoy; tamaño del iframe y uso en celular.
- Refresco programado, frecuencia de llegada de datos y volumen de `tb_Balance`.
- Entidades que se fusionan, cambian de nombre o se liquidan.
- Qué segmentadores casi no se usan.
- Un número de control del sector para validar el modelo.
- Formato de la documentación final y si hay más personas editando el informe.
- Si los cambios se editan directamente en los archivos `.tmdl` y `.json` o se guían desde Power BI Desktop.
- Qué no se debe tocar del diseño.
