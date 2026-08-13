# Caracterización y clasificación de ligandos inhibidores beta-secretasa 1 basados en el Potencial Electrostático Molecular 


Clasificación y regresión de afinidad de ligandos de la proteasa **BACE1** (Beta-secretasa 1,
diana terapéutica en Alzheimer), usando **K-Nearest Neighbors** y **K-Nearest Centroid** sobre **matrices de distancia**
construidas a partir de árboles/descriptores del **Potencial Electrostático Molecular (MEP)**
de cada molécula, en distintas combinaciones de su componente positiva y negativa.

Trabajo realizado en el **Grupo de Química Teórica** de la Universidad Nacional de
Colombia (UNAL), como proyecto de investigación tomado del **Grand Challenge 4 (GC4) — BACE** del D3R
(Drug Design Data Resource): <https://drugdesigndata.org/about/grand-challenge-4/bace>.

## Objetivo

1. Proponer y/o explorar herramientas alternativas para la caracterización y clasificación de los ligandos inhibidores BACE1 del GC4, así cómo la predicción de afinidades de unión de los mismos mediante la cuantificación de la similitud topológica, utilizando representaciones en forma de árboles de potencial electrostático molecular. 
2. Proponer valores de afinidad de unión para el conjunto de los ligandos inhibidores BACE1 cuya afinidad constituye el reto.

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

> Las matrices de distancia (`data/distance_matrix_neg`, etc.) no están incluidas en este repositorio


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

## Resultados

Ambos notebooks fueron ejecutados con todas las verificaciones de integridad correctas (173 IDs únicos, 153
moléculas válidas + 20 desconocidas, sin `mol116` en las matrices).

**Resumen del desempeño** (detalle completo en [`results/RESULTS.md`](./results/RESULTS.md)):

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

## Uso

```bash
git clone <url-del-repo>
cd <repo>
pip install -r requirements.txt
jupyter notebook notebooks/01_KNN_clasificacion_regresion.ipynb
```

O ábrelos directamente en Google Colab, activando `USAR_DRIVE = True` y ajustando `RUTA_BASE`
si tus matrices de distancia están en Drive.


## Licencia

Este proyecto está bajo licencia MIT — ver [`LICENSE`](./LICENSE).

## Créditos
Omar Landinez & Edgar Eduardo Daza Caicedo • 2026
Grupo de Química Teórica, Universidad Nacional de Colombia (UNAL). 
Datos del reto GC4 BACE1: [D3R Grand Challenge 4](https://drugdesigndata.org/about/grand-challenge-4/bace).
