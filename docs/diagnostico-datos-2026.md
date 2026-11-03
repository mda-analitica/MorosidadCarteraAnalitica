# Diagnóstico de datos y plan de corrección (modelo actualizado a agosto de 2026)

Fecha: 2026-11-03. Insumos revisados:
- Caché del modelo actualizado (`cache.abf`, datos de enero de 2025 a agosto de 2026).
- Archivos fuente de la Supersolidaria: `Supersolidaria_PUC_2025_08_31.xlsx` y `Supersolidaria_PUC_2026_08_31.xlsx`.

Convenciones: **[Verificado]** comprobado cruzando el modelo con los archivos fuente. **[Inferido]** es una hipótesis que falta confirmar.

---

## 1. Resumen

La caída de entidades en 2026 **no es real**: es un error de carga. El archivo de 2026 viene partido en dos hojas y el dataflow solo lee la primera, así que el informe pierde la mitad de las cooperativas. Esto hace que la cartera del sector aparezca a la mitad y la tasa de mora más baja de lo que es.

Con los datos completos, agosto de 2026 queda así:

| Agosto 2026 | Lo que muestra el modelo hoy | Lo correcto (dos hojas) |
|---|---|---|
| Entidades | 87 | 167 |
| Cartera (billones) | 9,50 | 20,95 |
| Tasa mora | 5,45 % | 5,86 % |
| Tasa riesgo | 6,92 % | 7,64 % |
| Riesgo/Mora | 1,27 | 1,30 |

No hay datos de octubre de 2026 en el modelo (el último cierre es agosto de 2026).

## 2. Entidades del alcance por mes (modelo actual)

| Cierre | Entidades | Cartera (billones) | Tasa mora | Tasa riesgo | Estado |
|---|---|---|---|---|---|
| Ene 2025 | 171 | 17,90 | 3,89 % | 8,76 % | Mora incompleta |
| Feb 2025 | 171 | 18,14 | 6,59 % | 8,77 % | OK |
| Mar 2025 | 172 | 18,28 | 6,28 % | 8,89 % | OK |
| Abr 2025 | 167 | 18,20 | 6,39 % | 8,73 % | Faltan 5 |
| May 2025 | 172 | 18,54 | 6,18 % | 8,59 % | OK |
| Jun 2025 | 172 | 18,63 | 6,45 % | 9,04 % | OK |
| Jul 2025 | 172 | 18,82 | 6,49 % | 8,73 % | OK |
| Ago 2025 | 172 | 18,95 | 6,46 % | 8,66 % | OK, cuadra al peso con el Excel |
| Sep 2025 | 10 | 0,96 | 5,15 % | 6,24 % | Inválido |
| Oct 2025 | 172 | 19,35 | 6,25 % | 8,33 % | OK |
| Nov 2025 | 171 | 19,49 | 6,18 % | 8,31 % | OK |
| Dic 2025 | 170 | 19,48 | 5,89 % | 7,97 % | OK |
| Ene 2026 | 93 | 9,15 | 5,09 % | 6,96 % | Falta la segunda hoja (probable) |
| Feb 2026 | 85 | 8,82 | 5,15 % | 6,89 % | Falta la segunda hoja (probable) |
| Mar 2026 | 27 | 4,27 | 4,78 % | 7,08 % | Inválido |
| Abr 2026 | 88 | 9,25 | 5,31 % | 6,94 % | Falta la segunda hoja (probable) |
| May 2026 | 87 | 9,27 | 5,41 % | 6,86 % | Falta la segunda hoja (probable) |
| Jun 2026 | 18 | 2,79 | 4,20 % | 6,46 % | Inválido |
| Jul 2026 | 87 | 9,44 | 5,47 % | 7,02 % | Falta la segunda hoja (probable) |
| Ago 2026 | 87 | 9,50 | 5,45 % | 6,92 % | **[Verificado]** falta la segunda hoja |

## 3. Problemas identificados

### P1. El dataflow solo lee la primera hoja del Excel (crítico)
- **[Verificado]** `Supersolidaria_PUC_2025_08_31.xlsx` tiene una sola hoja (`6DIG - AGOSTO`, 850.200 filas). `Supersolidaria_PUC_2026_08_31.xlsx` tiene dos (`Parte 1` y `Parte 2`, 457.470 filas cada una).
- **[Verificado]** De las 167 entidades del alcance presentes en el archivo de agosto de 2026, 87 están en `Parte 1` y 80 en `Parte 2`. El modelo tiene exactamente las 87 de `Parte 1`; las 80 de `Parte 2` faltan.
- **[Verificado]** Para las entidades que sí cargan, los valores de las 75 cuentas del catálogo coinciden al peso con el Excel (agosto de 2025 y agosto de 2026). El problema es de cobertura, no de valores.
- **[Inferido]** Enero, febrero, abril, mayo y julio de 2026 tienen el mismo patrón (85 a 93 entidades), así que probablemente también vienen en dos hojas.
- **[Verificado]** El archivo viene ordenado por código de entidad: `Parte 1` va del 6 al 2036 y `Parte 2` del 2036 al 20009.
- Impacto: como `Parte 2` trae las entidades con código más alto, la muestra no es aleatoria y sesga todas las cifras del sector, el ranking y la comparación con la entidad seleccionada.

### P2. Una entidad queda partida entre las dos hojas
- **[Verificado]** La entidad `2036` tiene la mitad de sus cuentas en `Parte 1` (hasta la 3310xx) y la otra mitad en `Parte 2`. Hoy no está en el alcance, pero el corte puede caer en una cooperativa del alcance otro mes. En ese caso cargaría la cartera (14xx) y perdería la mora (84xx), con una tasa de mora falsa en 0 %.

### P3. Meses con muy pocas entidades: septiembre 2025, marzo 2026 y junio 2026
- **[Verificado]** Septiembre 2025 tiene 10 a 11 entidades, 10.216 filas en grupos duplicados y valores negativos (entidad `2773`). Marzo 2026 tiene 27 entidades y junio 2026 tiene 18.
- **[Inferido]** Son demasiado pocas para explicarse por las dos hojas. Puede que esos archivos tengan otra estructura (más hojas, otro nombre de hoja, encabezados distintos) o que se haya cargado un archivo equivocado. Hace falta revisar esos tres archivos.

### P4. Enero 2025 tiene la mora incompleta
- **[Verificado]** La mora (cuentas 84xx) de enero es cerca de la mitad de la de febrero, mientras la cartera (14xx) es estable. La tasa de mora de enero (3,89 %) no es comparable. Hace falta revisar el archivo de enero de 2025.

### P5. Tipos de dato distintos entre archivos
- **[Verificado]** En 2025, `CODRENGLON` viene como texto (`"100000"`). En 2026 viene como número (`100000.0`) y `CODIGO ENTIDAD` como decimal (`6.0`). El modelo actual cruza bien, pero si el dataflow convierte el número a texto sin quitar el decimal (`"100000.0"`), las cuentas dejan de cruzar con `dm_Cuentas`. Conviene normalizarlo explícitamente.

### P6. El directorio de entidades es estático
- **[Verificado]** `dm_Entidades` es una lista fija de 174 entidades. En agosto de 2026 faltan 7 en el Excel: `1271` y `1376`, que nunca reportaron, y `13022`, `1450`, `374`, `4054` y `7099`, que reportaban en 2025.
- **[Verificado]** Hay 18 entidades en el archivo de 2026 que no estaban en el de 2025. Ninguna está en `dm_Entidades`, así que si alguna es una cooperativa de ahorro y crédito nueva, el informe no la ve.
- **[Inferido]** Las 5 que desaparecieron pueden haberse liquidado, fusionado o dejado de reportar. El modelo no tiene forma de saberlo (pendiente ya registrado).

### P7. Los meses incompletos no se distinguen en el informe
- El informe no avisa cuando un mes llegó incompleto. Un usuario que elija septiembre 2025 o junio 2026 ve cifras falsas sin ninguna advertencia. La medida nueva `Cantidad de entidades` ya permite detectarlo.

## 4. Plan de corrección

Orden propuesto: primero la fuente (dataflow), luego el modelo y al final el informe.

### Fase 1: dataflow (fuera del repo, lo hace el dueño)
1. **Leer todas las hojas de cada archivo**, no solo la primera. En Power Query, filtrar las hojas con `[Kind] = "Sheet"` y combinarlas antes de expandir. Esto corrige P1 y P2 juntos, porque al unir las hojas la entidad partida queda completa.
2. **Normalizar tipos:** convertir `CODIGO ENTIDAD` y `CODRENGLON` a entero y luego a texto, para que `6.0` quede `"6"` y `100000.0` quede `"100000"`. Corrige P5.
3. **Tomar la fecha del nombre del archivo** (`Supersolidaria_PUC_AAAA_MM_DD`) y no del contenido o de la hoja, si no se hace ya así.
4. **Quitar duplicados** por fecha, entidad y cuenta. Si un duplicado tiene valores distintos, registrarlo en vez de sumarlo.
5. **Revisar los archivos de septiembre 2025, marzo 2026, junio 2026 y enero 2025** para confirmar si el problema es de estructura o de la fuente.

Ejemplo de la combinación de hojas (adaptar el origen al que usa hoy el dataflow):

```powerquery
let
    Origen = Folder.Files("RUTA_DE_LOS_ARCHIVOS"),
    SoloPUC = Table.SelectRows(Origen, each Text.StartsWith([Name], "Supersolidaria_PUC_")),
    ConFecha = Table.AddColumn(SoloPUC, "Fecha reporte",
        each Date.FromText(Text.Replace(Text.Range([Name], 19, 10), "_", "-")), type date),
    ConHojas = Table.AddColumn(ConFecha, "Hojas",
        each Table.SelectRows(Excel.Workbook([Content], true), each [Kind] = "Sheet")),
    Hojas = Table.ExpandTableColumn(ConHojas, "Hojas", {"Name", "Data"}, {"Hoja", "Data"}),
    Datos = Table.ExpandTableColumn(Hojas, "Data",
        {"CODIGO ENTIDAD", "CODRENGLON", "VALOR EN PESOS"}),
    Tipos = Table.TransformColumns(Datos, {
        {"CODIGO ENTIDAD", each Text.From(Int64.From(_)), type text},
        {"CODRENGLON", each Text.From(Int64.From(_)), type text},
        {"VALOR EN PESOS", each Number.From(_), type number}}),
    Final = Table.Distinct(
        Table.SelectColumns(Tipos, {"Fecha reporte", "CODIGO ENTIDAD", "CODRENGLON", "VALOR EN PESOS"}),
        {"Fecha reporte", "CODIGO ENTIDAD", "CODRENGLON"})
in
    Final
```

### Fase 2: directorio de entidades
6. Actualizar `Entidades_PUC` con el listado vigente de cooperativas de ahorro y crédito de la Supersolidaria. Hay que revisar si alguna de las 18 entidades nuevas de 2026 pertenece al alcance.
7. Agregar a `dm_Entidades` el estado de la entidad (activa, liquidada, fusionada) y las fechas de inicio y fin de reporte, para que el informe sepa cuándo una entidad ya no debe esperarse.

### Fase 3: modelo semántico (lo hace Claude en el repo, con autorización)
8. **Control de cobertura:** agregar una medida de entidades esperadas frente a reportadas por mes, y una bandera de "mes incompleto" (por ejemplo, cobertura menor a 95 %).
9. **Proteger los indicadores del sector:** hacer que el informe muestre por defecto el último mes completo, y excluir o advertir los meses incompletos en las comparaciones.
10. **Tabla de calidad:** dejar una tabla o página oculta con entidades por mes y tipo, para revisar rápido después de cada carga.

### Fase 4: validación después de recargar
11. Comparar con los valores de referencia de este documento:
    - Agosto 2025: 172 entidades, cartera 18,95 billones, mora 6,46 %, riesgo 8,66 %.
    - Agosto 2026: 167 entidades, cartera 20,95 billones, mora 5,86 %, riesgo 7,64 %, Riesgo/Mora 1,30.
12. Revisar que ningún mes tenga menos de unas 165 entidades. Si alguno tiene menos, buscar la causa antes de publicar.

## 5. Lo que necesito para seguir
- Los archivos fuente de **septiembre 2025, marzo 2026, junio 2026 y enero 2025**, para diagnosticar P3 y P4.
- Saber cómo lee hoy el dataflow los archivos (carpeta, SharePoint o descarga directa, y si toma la primera hoja o una hoja por nombre), para ajustar el ejemplo de la Fase 1.
- Confirmar el comentario sobre **octubre de 2026**: el modelo no tiene ese mes. ¿Te referías a otro mes o a lo que se ve en la página nueva "Cantidad Entidades"?
