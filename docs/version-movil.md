# Versión móvil del informe (enlace público)

El enlace de "Publicar en la web" no aplica el diseño de teléfono de Power BI Desktop (solo la app móvil de Power BI lo muestra).
Por eso existe un informe aparte, vertical, que comparte el modelo semántico con el de escritorio.

## Archivos
- `MorosidadSectorSolidario_Movil.pbip` y `MorosidadSectorSolidario_Movil.Report/`: informe vertical. Su `definition.pbir` apunta a `../MorosidadSectorSolidario.SemanticModel` (un solo modelo).
- Generado con un script a partir del informe de escritorio; las posiciones salen de los mockups aprobados (ver abajo).

## Tamaños de página
| Página | Ancho × alto (px) | Modo |
|---|---|---|
| Inicio | 450 × 800 | Ajustar a la página, sin scroll |
| Por sector | 450 × 1730 | Ajustar al ancho (scroll vertical) |
| Por entidades | 450 × 1290 | Ajustar al ancho |
| Por entidad | 450 × 1570 | Ajustar al ancho |
| Cobertura de cartera | 450 × 1470 | Ajustar al ancho |

## Identidad (Manual de Identidad "Monitor GRC" v1.0)
Segoe UI; fondo `#F3F5F4`; tarjetas blancas con borde `#E1E5E3`; header oscuro en degradado (`#0C201A` → `#1F5646`); acento y botones `#008282`; verdes `#1B4B3D`, `#2A7560`, `#3AA082`, `#49CAA5`; una tarjeta oscura por página (cobertura).

## Decisiones respecto al informe de escritorio
- Se quitan los fondos de imagen, el panel de información y sus marcadores.
- Los segmentadores son listas desplegables con buscador; "Entidades a comparar" es de selección múltiple.
- "Línea de crédito" (Por entidades) pasa de segmentador avanzado a lista desplegable estándar.
- Medida nueva en el modelo: `Color barra seleccionada móvil` (teal para la entidad elegida, verde para las demás). La de escritorio no cambia.
- Pendiente de decisión: distintivo de estado (Bajo/Medio/Crítico) en la tarjeta de cobertura; no se incluyó.

## Qué revisar en Power BI Desktop
1. Abrir `MorosidadSectorSolidario_Movil.pbip` y actualizar.
2. En cada página, vista "Ajustar al ancho": confirmar que no se cortan títulos de segmentadores ni etiquetas.
3. Segmentador "Ver por" (Por sector): es un segmentador avanzado; revisar el color de su texto.
4. Segmentador de fechas: modo "Entre" (barra); si ocupa mucho, reducir el rango.
5. Tarjetas de indicadores: revisar el diseño en cuadrícula (2 columnas) y el tamaño de fuente.
6. Publicar el informe vertical conectado al conjunto de datos ya publicado (no duplicar el modelo) y probar el enlace desde un celular.
