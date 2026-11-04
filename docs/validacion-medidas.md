# Valores esperados de las medidas (corte 2026-08-31)

Calculados en Python con los datos del .pbix entregado el 2026-11-04 y el catálogo de `docs/referencia/catalogo-cuentas-modelo.csv`.
Sirven para comprobar el modelo en Power BI Desktop después de actualizar. Cifras en miles de millones de pesos.
Cómo comprobar: en una matriz de Power BI, ponga `dm_Cuentas[Modalidad]` (o `Línea`) en filas, filtre `Calendario[Fecha]` al corte y agregue las medidas base (`Saldo total`, `Cartera total cuenta 84`, `Tasa riesgo`, `Tasa mora`, `Riesgo/Mora`, `Deterioro total`, `Cobertura cartera en riesgo`, `Deterioro sobre cartera bruta`, `Cobertura individual CDE`). Para una entidad, filtre `dm_Entidades[CODIGO ENTIDAD]`.
En las filas por modalidad o línea, la cobertura usa solo el deterioro individual; en el total suma también el deterioro general.

### Sector (todas las entidades)

| Corte | Cartera bruta | Cartera 84 | Saldo en riesgo | Saldo en mora | Tasa riesgo | Tasa mora | Riesgo/Mora | Deterioro total | Cobertura | Det./bruta | Cobertura CDE |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Total | 20.954,4 | 20.954,4 | 1.600,8 | 1.228,6 | 7,64 % | 5,86 % | 1,30 | 1.502,1 | 93,83 % | 7,17 % | 63,01 % |
| Modalidad Vivienda | 968,0 | 967,1 | 34,3 | 27,3 | 3,54 % | 2,82 % | 1,26 | 6,4 | 18,59 % | 0,66 % | 27,45 % |
| Modalidad Consumo | 15.269,6 | 15.270,5 | 1.200,6 | 904,9 | 7,86 % | 5,93 % | 1,33 | 645,8 | 53,79 % | 4,23 % | 63,01 % |
| Modalidad Microcrédito | 2.257,2 | 2.000,8 | 156,5 | 128,3 | 6,93 % | 6,41 % | 1,08 | 115,9 | 74,05 % | 5,13 % | 69,72 % |
| Modalidad Comercial | 2.459,5 | 2.716,0 | 209,4 | 168,2 | 8,51 % | 6,19 % | 1,37 | 116,2 | 55,51 % | 4,73 % | 63,01 % |
| Línea Consumo con libranza | 7.303,6 | 7.258,3 | 206,6 | 117,2 | 2,83 % | 1,61 % | 1,75 | 0,0 |  |  |  |
| Línea Consumo sin libranza | 7.966,0 | 8.012,1 | 994,0 | 787,7 | 12,48 % | 9,83 % | 1,27 | 0,0 |  |  |  |
| Producto Productivo | 2.152,1 | 0,0 | 118,3 | 0,0 | 5,50 % |  |  | 84,7 | 71,58 % | 3,94 % | 62,62 % |

### Entidad 3246

| Corte | Cartera bruta | Cartera 84 | Saldo en riesgo | Saldo en mora | Tasa riesgo | Tasa mora | Riesgo/Mora | Deterioro total | Cobertura | Det./bruta | Cobertura CDE |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Total | 1.192,5 | 1.192,5 | 50,9 | 29,1 | 4,27 % | 2,44 % | 1,75 | 58,8 | 115,66 % | 4,93 % | 70,94 % |
| Modalidad Vivienda | 1,5 | 1,5 | 0,0 | 0,0 | 0,00 % | 0,00 % |  | 0,0 |  | 0,02 % |  |
| Modalidad Consumo | 624,9 | 624,9 | 25,5 | 14,5 | 4,08 % | 2,33 % | 1,76 | 14,3 | 55,95 % | 2,28 % | 69,40 % |
| Modalidad Microcrédito | 446,9 | 219,5 | 15,6 | 7,3 | 3,49 % | 3,34 % | 1,04 | 18,8 | 120,99 % | 4,22 % | 85,80 % |
| Modalidad Comercial | 119,2 | 346,6 | 9,8 | 7,2 | 8,20 % | 2,08 % | 3,95 | 3,8 | 39,26 % | 3,22 % | 52,52 % |

### Entidad 8024

| Corte | Cartera bruta | Cartera 84 | Saldo en riesgo | Saldo en mora | Tasa riesgo | Tasa mora | Riesgo/Mora | Deterioro total | Cobertura | Det./bruta | Cobertura CDE |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Total | 2.212,8 | 2.212,8 | 201,1 | 147,7 | 9,09 % | 6,68 % | 1,36 | 298,6 | 148,46 % | 13,49 % | 73,43 % |
| Modalidad Vivienda | 1,3 | 1,3 | 0,3 | 0,3 | 21,61 % | 21,15 % | 1,02 | 0,1 | 51,81 % | 11,20 % | 52,93 % |
| Modalidad Consumo | 1.155,6 | 1.155,6 | 121,2 | 80,7 | 10,48 % | 6,98 % | 1,50 | 79,1 | 65,27 % | 6,84 % | 72,37 % |
| Modalidad Microcrédito | 673,9 | 673,9 | 49,2 | 41,4 | 7,31 % | 6,14 % | 1,19 | 38,2 | 77,66 % | 5,67 % | 72,66 % |
| Modalidad Comercial | 381,9 | 381,9 | 30,4 | 25,4 | 7,97 % | 6,65 % | 1,20 | 23,4 | 76,93 % | 6,13 % | 78,74 % |

### Entidad 1442

| Corte | Cartera bruta | Cartera 84 | Saldo en riesgo | Saldo en mora | Tasa riesgo | Tasa mora | Riesgo/Mora | Deterioro total | Cobertura | Det./bruta | Cobertura CDE |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Total | 45,7 | 45,7 | 2,7 | 2,4 | 5,80 % | 5,14 % | 1,13 | 1,8 | 69,06 % | 4,00 % | 26,82 % |
| Modalidad Vivienda | 2,5 | 2,5 | 0,0 | 0,0 | 0,00 % | 0,00 % |  | 0,0 |  | 0,00 % |  |
| Modalidad Consumo | 41,8 | 42,1 | 2,6 | 2,3 | 6,33 % | 5,57 % | 1,14 | 0,7 | 27,93 % | 1,77 % | 26,84 % |
| Modalidad Microcrédito | 0,1 | 0,1 | 0,0 | 0,0 | 2,72 % | 2,72 % | 1,00 | 0,0 | 18,05 % | 0,49 % | 0,00 % |
| Modalidad Comercial | 1,3 | 1,0 | 0,0 | 0,0 | 0,00 % | 0,00 % |  | 0,0 |  | 0,06 % |  |

### Serie mensual del sector

| Corte | Cartera bruta | Cartera 84 | Saldo en riesgo | Saldo en mora | Tasa riesgo | Tasa mora | Riesgo/Mora | Deterioro total | Cobertura | Det./bruta | Cobertura CDE |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 2025-10-31 | 19.349,9 | 19.349,9 | 1.612,6 | 1.209,6 | 8,33 % | 6,25 % | 1,33 | 1.419,9 | 88,05 % | 7,34 % | 61,55 % |
| 2025-11-30 | 19.491,1 | 19.491,1 | 1.620,4 | 1.204,0 | 8,31 % | 6,18 % | 1,35 | 1.427,7 | 88,11 % | 7,32 % | 61,61 % |
| 2025-12-31 | 19.481,0 | 19.481,1 | 1.552,3 | 1.146,6 | 7,97 % | 5,89 % | 1,35 | 1.401,7 | 90,30 % | 7,20 % | 61,48 % |
| 2026-01-31 | 19.548,4 | 19.548,5 | 1.591,9 | 1.172,4 | 8,14 % | 6,00 % | 1,36 | 1.422,1 | 89,33 % | 7,27 % | 61,51 % |
| 2026-02-28 | 19.538,0 | 19.538,2 | 1.571,6 | 1.160,7 | 8,04 % | 5,94 % | 1,35 | 1.410,0 | 89,72 % | 7,22 % | 61,79 % |
| 2026-03-31 | 20.024,9 | 20.025,1 | 1.612,7 | 1.213,2 | 8,05 % | 6,06 % | 1,33 | 1.451,9 | 90,03 % | 7,25 % | 62,27 % |
| 2026-04-30 | 20.164,5 | 20.164,5 | 1.594,2 | 1.196,4 | 7,91 % | 5,93 % | 1,33 | 1.458,5 | 91,49 % | 7,23 % | 62,29 % |
| 2026-05-31 | 20.390,8 | 20.390,8 | 1.578,8 | 1.193,3 | 7,74 % | 5,85 % | 1,32 | 1.455,1 | 92,16 % | 7,14 % | 61,60 % |
| 2026-06-30 | 20.600,8 | 20.600,9 | 1.619,4 | 1.209,7 | 7,86 % | 5,87 % | 1,34 | 1.474,0 | 91,02 % | 7,16 % | 61,71 % |
| 2026-07-31 | 20.771,7 | 20.732,6 | 1.593,7 | 1.202,4 | 7,67 % | 5,80 % | 1,32 | 1.486,5 | 93,28 % | 7,16 % | 62,80 % |
| 2026-08-31 | 20.954,4 | 20.954,4 | 1.600,8 | 1.228,6 | 7,64 % | 5,86 % | 1,30 | 1.502,1 | 93,83 % | 7,17 % | 63,01 % |

### Entidades con datos por corte

| Corte | Entidades |
|---|---|
| 2025-10-31 | 172 |
| 2025-11-30 | 171 |
| 2025-12-31 | 170 |
| 2026-01-31 | 170 |
| 2026-02-28 | 164 |
| 2026-03-31 | 169 |
| 2026-04-30 | 168 |
| 2026-05-31 | 169 |
| 2026-06-30 | 169 |
| 2026-07-31 | 167 |
| 2026-08-31 | 167 |
