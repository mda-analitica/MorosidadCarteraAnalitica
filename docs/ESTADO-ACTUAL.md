# Estado actual y siguiente paso — Informe móvil

> Documento de traspaso. Lo lee una persona o IA **sin contexto previo**. Explica qué es el proyecto,
> qué se hizo de último y qué sigue. Fecha: 11 nov. 2026. Rama de trabajo: `claude/dazzling-albattani-vshphc`.
> Último commit: `73233c5` ("#1 Filtros ocultos en Inicio con páginas de filtro separadas").

## 1. Qué es el proyecto
Informe de Power BI (**MorosidadSectorSolidario**) sobre morosidad, riesgo y cobertura de la cartera de
las cooperativas de ahorro y crédito de Colombia, con datos públicos de la **Supersolidaria** (cuenta 14 =
cartera por calificación A–E; cuenta 84 = cartera por días de mora). Los datos entran por un flujo de datos
(dataflow) de Power BI; el corte vigente es **agosto de 2026**.

Hay **dos informes** que comparten la lógica pero **tienen cada uno su propio modelo semántico** (decisión
del usuario; se unificarán después):
- **Escritorio:** `MorosidadSectorSolidario.pbip` / `.Report` / `.SemanticModel`.
- **Móvil (vertical, para el enlace "Publicar en la web"):** `MorosidadSectorSolidario_Movil.pbip` /
  `.Report` / `.SemanticModel`. **Todo el trabajo reciente es sobre el móvil.**

Motivo del informe móvil: el diseño de teléfono de Power BI **no** se aplica en "Publicar en la web"; por eso
se hizo un informe aparte con páginas verticales de **450 px de ancho**.

## 2. Cómo se trabaja aquí (importante)
- **No hay Power BI Desktop en este entorno.** No se puede abrir ni renderizar el `.pbix`/`.pbip`. Solo se
  editan los archivos PBIR (JSON de visuales) y TMDL (modelo), y se valida la **estructura** (JSON válido,
  sin visuales fuera de página, navegación e interacciones que apunten a objetos existentes, medidas que
  existan). **El resultado visual siempre lo verifica el usuario en su Desktop.**
- Los archivos JSON del informe se escriben con **CRLF** y `json.dumps(indent=2, ensure_ascii=False)` con
  `\n` reemplazado por `\r\n`. Los `.tmdl` también en CRLF. Las medidas nuevas se agregan **al final** de
  `Medidas.tmdl`, antes de `partition Medidas`.
- Flujo de entrega: editar → validar estructura → `git commit`/`push` a la rama →
  `git archive` a un `.zip` en `entrega/` (carpeta ignorada por git) → enviar el zip al usuario para que lo
  abra en Desktop, actualice y guarde como `.pbix`.
- **Marca "Monitor GRC"** (ver `docs` del repo de identidad): tipografía Segoe UI; fondo `#F3F5F4`;
  tarjetas blancas con borde `#E1E5E3`; encabezado oscuro en degradado (`#0C201A`→`#1F5646`); acento y
  botones teal `#008282`; verdes `#1B4B3D`, `#2A7560`, `#3AA082`, `#49CAA5`.
- **Regla de trabajo del usuario:** antes de implementar una pantalla, se le muestra un **mockup** (PNG hecho
  con HTML + Playwright a 450 px, con cifras reales) y se espera su aprobación.

## 3. Páginas del informe móvil (ids)
| Página | id |
|---|---|
| Inicio | `e95937ff432803bce94e` |
| Por sector | `f18b5d9b33d8b122409c` |
| Por entidades (Benchmarking) | `55e96498ed829bce84f1` |
| Por entidad | `b3cfda54146a6fa53763` |
| Cobertura de cartera | `cb83cc2dc15c0e00bbfe` |
| Glosario (oculta) | `5d51dde2eba4df968358` |
| Benchmarking · filtros (oculta) | `a5552ac678c56f67e1de` |
| Entidad · filtros (oculta) | `3c7e3ae7b577e3a6c1a0` |
| Cobertura · filtros (oculta) | `0593e6e8d94d3ef6abf7` |
| Tooltip-Mora / Tooltip-Riesgo (ocultas) | `5849a7101d8b824365e3` / `f37d073b3d29f68ca814` |

## 4. Mejoras pedidas (11 requisitos) y su estado
| # | Requisito | Estado |
|---|---|---|
| 1 | Filtros ocultos en Inicio | **Hecho (lo último)**, falta que el usuario lo pruebe |
| 2 | Definición de indicadores (Glosario + botones "i") | Hecho |
| 3 | Detalle al tocar una barra en Por entidad | Hecho |
| 4 | Ranking en Por entidades | Hecho |
| 5 | Selector de indicador en la línea de cobertura | Hecho |
| 6 | Sincronizar filtros entre páginas | Hecho |
| 7 | Fechas visibles (corte + Supersolidaria) | Corte: hecho. **Supersolidaria: pendiente** |
| 8 | Histórico de actualizaciones | **Fuera de alcance** por ahora |
| 9 | KPIs de cobertura: 7 tarjetas | Hecho |
| 10 | Tooltips en verde | Hecho |
| 11 | Siempre el último corte (sin selector de fecha) | Hecho |

Detalle y decisiones en `docs/plan-mejoras-movil.md`.

## 5. Lo ÚLTIMO que se hizo: #1 Filtros ocultos en Inicio (commit `73233c5`)
Implementado con **páginas de filtro separadas** (navegación por botones), **no con marcadores**, porque la
navegación entre páginas es más confiable en el enlace publicado.
- **Inicio** ya no muestra segmentadores: solo el encabezado (con fecha de corte y de Supersolidaria) y 4
  botones.
- **"Indicadores por sector"** navega directo a Por sector.
- **"Benchmarking"**, **"Informe por entidad"** y **"Cobertura"** abren cada uno su página de filtros
  (oculta), con botón **"Volver"** a Inicio y botón **"Ver…"** a la página de informe.
- **Benchmarking · filtros:** paso 1 "Entidades a comparar" (lista, selección múltiple); paso 2 "Su entidad"
  (lista, selección única) **limitada a las elegidas en el paso 1** mediante la medida nueva
  `Entidad está en comparadas` (en `Medidas.tmdl` del modelo móvil), aplicada como filtro de ese segmentador.
- **Entidad · filtros** y **Cobertura · filtros:** solo "Su entidad" (lista); "Entidades a comparar" aparece
  como recuadro bloqueado "No aplica en este informe".
- Los dos filtros de entidad usan el mismo estilo de lista (`listSlicer`) y están **sincronizados** con las
  páginas de informe: Su entidad → grupo `SIGLA`; Entidades a comparar → grupo `SIGLA1`. Así la selección
  viaja a las páginas de informe.
- Las 3 páginas de filtro están marcadas `"visibility": "HiddenInViewMode"` en su `page.json` para no
  aparecer en la navegación del informe publicado.

Entrega enviada: `entrega/MorosidadSectorSolidario_Movil_req1_2026-11-11.zip`.

### Qué debe verificar el usuario en Power BI Desktop (en orden de riesgo)
1. **Paso 2 limitado al paso 1** (lo más delicado): que en Benchmarking la lista "Su entidad" solo muestre
   las entidades marcadas en "Entidades a comparar". Si falla, revisar la medida `Entidad está en comparadas`
   y su filtro de visual.
2. **Activar "Buscar"** en cada lista con el botón ⋯ del visual (no se dejó activado por archivo para no
   arriesgar el visual).
3. **Selección única** real en "Su entidad".
4. **Navegación** de los botones y de "Volver"/"Ver…", en Desktop y en el enlace publicado.
5. Que las **páginas de filtro no aparezcan** en la barra de páginas del informe publicado.

## 6. SIGUIENTE PASO
1. **Esperar la verificación del usuario del #1** en Desktop (los 5 puntos de arriba) y ajustar lo que falle,
   sobre todo el filtro del paso 2.
2. **Cerrar el #7 (fecha de la Supersolidaria)**, que quedó pendiente:
   - La fecha a mostrar es **"Fecha de Última Actualización"** de
     `https://www.supersolidaria.gov.co/es/content/entidades-vigiladas-que-reportan-informacion`,
     pestaña "Estados Financieros 6 Dígitos / 2026" (en la captura del usuario: **19/10/2026**).
   - **Este entorno no puede abrir ese dominio** (proxy 403). El usuario debe correr en su máquina el script
     de Python (lógica en `docs/plan-mejoras-movil.md`): descarga la página, dice si la fecha está en el HTML
     o la pinta JavaScript, y extrae la fecha. **Si está en el HTML**, la consulta de Power Query
     `Fecha Supersolidaria` (ya existe en el modelo móvil, ver `docs/plan-mejoras-movil.md`) funcionará; **si
     no**, se reemplaza por una tabla escrita a mano que el usuario actualiza cada mes.
3. Pendientes de fondo (no pedidos aún): **unificar los dos modelos** (escritorio y móvil) y el **#8
   histórico de actualizaciones** (necesita almacenamiento persistente fuera de Power BI).

## 7. Documentos de referencia en el repo
- `docs/plan-mejoras-movil.md` — plan de las 11 mejoras, decisiones del usuario y avance.
- `docs/version-movil.md` — cómo se construyó el informe vertical.
- `docs/diccionario-de-datos.md` — definición de cada indicador y de las cuentas 14 y 84.
- `docs/validacion-medidas.md` — valores esperados por corte para comprobar el modelo en Desktop.
- `docs/referencia/catalogo-cuentas-modelo.csv` — catálogo de cuentas usado por el modelo.
