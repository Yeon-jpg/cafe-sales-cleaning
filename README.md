# Limpieza de datos — Cafe Sales (Dirty Data)

Práctica individual de limpieza y transformación de datos con Python, aplicada al dataset [**Cafe Sales – Dirty Data for Cleaning Training**](https://www.kaggle.com/datasets/ahmedmohamed2003/cafe-sales-dirty-data-for-cleaning-training) (Kaggle), diseñado intencionalmente para contener problemas de calidad de datos.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `limpieza_cafe_sales.ipynb` | Notebook con todo el proceso: carga, inspección, diagnóstico, limpieza/transformación justificada, verificación y exportación. |
| `data/dirty_cafe_sales.csv` | Base de datos original (sucia), tal como se descargó de Kaggle. |
| `data/cafe_sales_clean.csv` | Base de datos resultante después de la limpieza. |
| `tabla_resumen_limpieza.csv` | Tabla resumen de problemas encontrados, registros afectados, acción realizada y justificación. |

## Resumen del procedimiento

1. **Carga e inspección**: se cargó el CSV con Pandas y se revisaron tipos de datos, valores únicos por columna y estadísticas descriptivas. Se detectó que todas las columnas se leían como texto y que existían dos marcadores de dato inválido usados como texto (`'ERROR'`, `'UNKNOWN'`), además de nulos reales.

2. **Diagnóstico de problemas de calidad**: se cuantificó, por columna, cuántos valores eran `NaN`, `'ERROR'` o `'UNKNOWN'`, y se verificó la existencia de duplicados (no se encontraron).

3. **Limpieza y transformación**:
   - Se unificaron `'ERROR'`/`'UNKNOWN'` como `NaN` para tener un único criterio de dato faltante.
   - Se corrigieron los tipos de datos (`Quantity`, `Price Per Unit`, `Total Spent` a numérico; `Transaction Date` a fecha).
   - Se aprovechó que **cada producto tiene un precio unitario fijo** y que siempre se cumple `Total Spent = Quantity × Price Per Unit` en los registros completos, para **reconstruir de forma algebraica** (no estadística) los valores faltantes de `Item`, `Quantity`, `Price Per Unit` y `Total Spent` cuando había suficiente información relacionada.
   - Los pocos valores que no pudieron reconstruirse sin ambigüedad se dejaron como nulos (marcados con la columna `Venta_incompleta`) en lugar de inventarse con la media o la moda.
   - `Payment Method` y `Location`, con un porcentaje muy alto de nulos (~32 % y ~40 %), se sustituyeron por la categoría explícita `"Desconocido"` en vez de imputarse o eliminar las filas, para no perder el resto de la información de venta ni sesgar el análisis de método de pago/canal.
   - Las fechas faltantes se dejaron como `NaT`: no existe forma de inferirlas sin inventar información.

4. **Verificación de duplicados y valores atípicos**: se confirmó que no había filas duplicadas y que todos los valores numéricos y de fecha caían dentro de los rangos de negocio esperados (cantidades de 1 a 5, precios según catálogo, fechas dentro de 2023), por lo que no se aplicó ningún filtro de outliers.

5. **Exportación**: la base limpia se guardó en `data/cafe_sales_clean.csv`, y la tabla de decisiones en `tabla_resumen_limpieza.csv`.

## Tabla resumen de problemas y decisiones

Ver [`tabla_resumen_limpieza.csv`](./tabla_resumen_limpieza.csv) para el detalle completo. En resumen:

| Problema encontrado | Registros afectados | Acción realizada | Justificación (resumen) |
|---|---|---|---|
| Marcadores `'ERROR'`/`'UNKNOWN'` | 3,256 celdas | Reemplazo global por `NaN` | Unificar el criterio de dato faltante antes de limpiar |
| Tipos de datos incorrectos | 4 columnas (10,000 filas) | Conversión a numérico/fecha | Habilitar operaciones aritméticas y de fecha |
| Faltantes en `Item` | 969 | Recuperado por precio único; resto → "Desconocido" | Evitar adivinar cuando el precio es ambiguo |
| Faltantes en `Price Per Unit` | 533 | Recuperado por catálogo y por Total/Quantity; 6 sin recuperar | El precio por producto es constante: inferencia segura |
| Faltantes en `Quantity` | 479 | Reconstruido como Total/Precio; 23 sin recuperar | Relación algebraica verificada al 100% en datos completos |
| Faltantes en `Total Spent` | 502 | Reconstruido como Cantidad×Precio; 23 sin recuperar | Misma relación algebraica |
| Duplicados | 0 | Verificado, sin acción | No se encontraron duplicados |
| Faltantes en `Payment Method` | 3,178 | → "Desconocido" | Demasiados nulos para imputar sin sesgar |
| Faltantes en `Location` | 3,961 | → "Desconocido" | Mismo criterio que Payment Method |
| Faltantes en `Transaction Date` | 460 | Se dejó como `NaT` | No hay forma de inferir la fecha real |
| Filas totalmente irrecuperables | 0 | Ninguna (no hubo casos) | Umbral definido pero no necesario |
| Valores atípicos | 0 detectados | No se aplicó filtro | Todos los valores están en rango de negocio esperado |

## Reflexión

No todos los datos "anómalos" se eliminaron o modificaron automáticamente: se priorizó reconstruir valores mediante relaciones reales del negocio (catálogo de precios, `Total = Cantidad × Precio`) sobre imputar con estadísticos genéricos, y se prefirió dejar una categoría explícita `"Desconocido"` o un valor nulo documentado antes que inventar un dato que pudiera sesgar el análisis posterior.

## Cómo reproducir

```bash
pip install pandas numpy jupyter
jupyter notebook limpieza_cafe_sales.ipynb
```

## Declaración de uso de IA

Durante la preparación de este repositorio se utilizó Claude (Anthropic) como apoyo para estructurar el notebook, redactar las justificaciones de cada decisión de limpieza y generar el README. El código fue revisado y ejecutado para verificar sus resultados antes de su entrega.
