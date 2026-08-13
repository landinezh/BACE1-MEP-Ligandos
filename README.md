# BACE1 · GC4 (D3R) — Predicción de afinidad con kNN sobre matrices de distancia de MEP

Clasificación y regresión de afinidad de ligandos de la proteasa **BACE1** (Beta-secretasa 1,
diana terapéutica en Alzheimer), usando **K-Nearest Neighbors** sobre **matrices de distancia**
construidas a partir de árboles/descriptores del **Potencial Electrostático Molecular (MEP)**
de cada molécula, en distintas combinaciones de su componente positiva y negativa.

Trabajo realizado en el semillero de **Química Teórica** de la Universidad Nacional de
Colombia (UNAL), como aporte al **Grand Challenge 4 (GC4) — BACE** del D3R
(Drug Design Data Resource): <https://drugdesigndata.org/about/grand-challenge-4/bace>.

## Objetivo

1. Usar la matriz de distancias derivada del MEP para **clasificar y encontrar patrones**
   entre moléculas con afinidad experimental conocida.
2. **Predecir la afinidad** de 20 moléculas de pose del GC4 que no cuentan con dato
   experimental, como aporte a uno de los objetivos del reto GC4 BACE1.

## Datos

| Archivo | Descripción |
|---|---|
| `data/BACE_affinity.csv` | ID original (`BACE_N`), SMILES, afinidad experimental e `ID_real` (nomenclatura interna `molN`) de las 154 moléculas con afinidad conocida. |
| `distance_matrix_neg` * | Matriz de distancias — componente 100% negativo del MEP. |
| `distance_matrix_pos_col` * | Matriz de distancias — componente 100% positivo del MEP. |
| `mix_50_50.csv` * | Matriz de distancias combinada 50% negativo / 50% positivo. |
| `mix_30_70.csv` * | Matriz de distancias combinada 30% negativo / 70% positivo. |
| `mix_70_30.csv` * | Matriz de distancias combinada 70% negativo / 30% positivo. |

\* Archivos generados en el paso previo de procesamiento de MEP (no incluidos en este
repositorio por su tamaño). Colócalos en `data/` o ajusta `RUTA_BASE` en los notebooks para
leerlos desde tu Google Drive si trabajas en Colab.

**Notas sobre los datos:**
- De las 154 moléculas con afinidad conocida (`mol0`–`mol153`), **`mol116` no está incluida
  en las matrices de distancia**: su geometría estaba cargada (ion), lo que hacía "explotar"
  el cálculo del MEP. Por eso el conjunto de entrenamiento/evaluación efectivo es de 153
  moléculas.
- Hay 20 moléculas adicionales de *pose* del GC4 (`xyz_newmol0`...`xyz_newmol19`) sin
  afinidad experimental conocida — son el objetivo de predicción de este proyecto.

## Estructura del repositorio

```
.
├── data/
│   ├── BACE_affinity.csv               # tabla de afinidades (SMILES, ID_real, etc.)
│   ├── distance_matrix_neg             # matriz de distancias, componente MEP negativo
│   ├── distance_matrix_pos_col         # matriz de distancias, componente MEP positivo
│   ├── mix_50_50.csv                   # matriz combinada 50% negativo / 50% positivo
│   ├── mix_30_70.csv                   # matriz combinada 30% negativo / 70% positivo
│   └── mix_70_30.csv                   # matriz combinada 70% negativo / 30% positivo
├── notebooks/
│   ├── 01_KNN_clasificacion_regresion.ipynb         # barrido de K (método del codo) + dendrogramas — YA EJECUTADO
│   └── 02_kNC_clasificacion_regresion_modular.ipynb # pipeline modular por funciones, K=5 fijo — YA EJECUTADO
├── results/
│   ├── RESULTS.md                      # resumen de resultados y lectura honesta del desempeño
│   ├── figuras/                        # matrices de confusión, elbow plots, dispersión, dendrogramas
│   └── tablas/                         # resumen de rendimiento y predicciones por matriz (CSV)
├── requirements.txt
├── LICENSE                             # MIT
└── README.md
```

> Las matrices de distancia (`data/distance_matrix_neg`, etc.) sí están incluidas en este
> repositorio: son livianas (150–750 KB cada una) y son indispensables para reproducir los
> resultados sin depender de Google Drive.

## Metodología

Para cada una de las 5 matrices de distancia:

1. **Limpieza y normalización** de la matriz (min-max a `[0, 1]`).
2. **Clasificación** (`KNeighborsClassifier`, `metric='precomputed'`) usando 5 categorías de
   afinidad (codificadas por color, ver `get_affinity_color` en los notebooks).
3. **Regresión** (`KNeighborsRegressor`, `metric='precomputed'`) sobre el valor continuo de
   afinidad.
4. **Evaluación** con accuracy / matriz de confusión / reporte de clasificación (clasificación)
   y MSE / R² (regresión), sobre un conjunto de prueba separado de las moléculas conocidas.
5. **Predicción** de clase y valor de afinidad para las 20 moléculas de pose sin dato
   experimental.
6. *(Notebook 01)* **Dendrogramas coloreados** por categoría de afinidad, para inspeccionar
   visualmente agrupaciones/patrones en el espacio de distancias de MEP.

## Correcciones realizadas sobre la versión original de los notebooks

- **Identificadores duplicados**: la limpieza original de etiquetas colapsaba las 20
  moléculas de pose (`xyz_newmol0`...`xyz_newmol19`) a un único label duplicado `"xyz"`,
  lo que rompía la indexación por `.loc[]`. Se corrigió para conservar identificadores
  únicos.
- **Categorías de afinidad con huecos**: los rangos de `get_affinity_color` no eran
  contiguos entre categorías vecinas; se cerraron los huecos para evitar una sexta clase
  "por defecto" no documentada.
- **`ID_real` no coincidía en formato** con lo que el resto del código esperaba
  (`"molN"` vs. entero plano); se añadió una normalización explícita.
- Se **removieron celdas duplicadas** (método del codo repetido dos veces).
- Se **evitó recalcular** el `train_test_split` en cada iteración del barrido de K
  (mismo resultado, cómputo redundante).
- Se **añadieron** las gráficas de dendrograma que usan las funciones de coloreado
  jerárquico, definidas en el notebook original pero nunca invocadas.
- Se **eliminó la dependencia obligatoria de Google Drive**: los notebooks ahora leen por
  defecto desde `data/` dentro del repo, con una bandera para montar Drive si se prefiere.

## Resultados

Ambos notebooks fueron **ejecutados de punta a punta contra los datos reales**, sin
errores, con todas las verificaciones de integridad correctas (173 IDs únicos, 153
moléculas válidas + 20 desconocidas, `mol116` confirmado ausente de las matrices).

**Resumen honesto del desempeño** (detalle completo en [`results/RESULTS.md`](./results/RESULTS.md)):

- **Clasificación**: accuracy entre 0.24–0.41 según la matriz y K, apenas en el rango del
  baseline de clase mayoritaria (0.294). La mejor combinación observada es `Mixed 50-50`
  con K=15 (accuracy=0.413).
- **Regresión**: **R² negativo en todas las combinaciones evaluadas** — el modelo no logra
  superar a la media como predictor. `Mixed 30-70` es la matriz más estable (R²≈-0.06,
  casi neutra); `Negative Distance` es la más inestable.
- Los **dendrogramas** (`results/figuras/dendrograma_*.png`) muestran que la mayoría de
  clústeres mezclan categorías de afinidad (ramas grises), consistente con la clasificación
  débil.
- Las **predicciones para las 20 moléculas de pose del GC4** están en
  `results/tablas/predicciones_*.csv`, pero dado el desempeño de validación deben tratarse
  como una primera aproximación, no como resultado final — ver recomendaciones en
  `results/RESULTS.md`.

Este desempeño limitado de la línea base de kNN es precisamente lo que motiva la fase de
transfer learning descrita más abajo.

## Uso

```bash
git clone <url-del-repo>
cd <repo>
pip install -r requirements.txt
jupyter notebook notebooks/01_KNN_clasificacion_regresion.ipynb
```

O ábrelos directamente en Google Colab, activando `USAR_DRIVE = True` y ajustando `RUTA_BASE`
si tus matrices de distancia están en Drive.

## Próximos pasos

Se está explorando una fase de **transfer learning** usando la matriz de distancias y los
árboles de MEP como entrada de una GCNN preentrenada (p. ej. DeepBindGCN) para la predicción
de interacción proteína–ligando, buscando mejorar sobre la línea base de kNN sin incurrir en
sobreajuste. Este trabajo se documentará en una rama o repositorio aparte una vez esté listo.

## Licencia

Este proyecto está bajo licencia MIT — ver [`LICENSE`](./LICENSE).

## Créditos

Semillero de Química Teórica, Universidad Nacional de Colombia (UNAL).
Datos del reto GC4 BACE1: [D3R Grand Challenge 4](https://drugdesigndata.org/about/grand-challenge-4/bace).
