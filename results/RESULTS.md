# Resultados — BACE1 GC4, kNN/kNC sobre matrices de distancia de MEP

Resultados obtenidos al ejecutar los dos notebooks (`01_KNN_clasificacion_regresion.ipynb`
y `02_kNC_clasificacion_regresion_modular.ipynb`) contra los datos reales:
153 moléculas conocidas (`mol0`–`mol153`, sin `mol116`) y 20 moléculas de pose del GC4
(`xyz_newmol0`–`xyz_newmol19`), sobre las 5 matrices de distancia derivadas del MEP.

## 2. Clasificación — barrido de K (notebook 01, K=1..20)

Accuracy sobre el conjunto de prueba (30%), con reporte detallado en K=3. Como referencia,
predecir siempre la clase mayoritaria ("Rojo Vino", 45/153 moléculas) da **accuracy = 0.294**.

| Matriz de distancia | Accuracy (K=3) | Mejor accuracy | Mejor K |
|---|---|---|---|
| Negative Distance | 0.261 | 0.326 | K=17 |
| Positive Column Distance | 0.304 | 0.370 | K=6 |
| Mixed 50-50 | 0.283 | 0.413 | K=15 |
| Mixed 30-70 | 0.370 | 0.370 | K=3 |
| Mixed 70-30 | 0.239 | 0.370 | K=14 |

Ver `figuras/elbow_accuracy_todas_matrices.png` y las matrices de confusión individuales
(`figuras/confusion_matrix_*.png`).

Ninguna matriz supera de forma consistente y clara al baseline de
clase mayoritaria (0.294). La mejor combinación observada es **Mixed 50-50 con K=15**
(accuracy=0.413), pero la mayoría de configuraciones se mueven en un rango de 0.24–0.37,
muy cerca del azar ponderado por clase. Las matrices de confusión muestran confusión
sistemática entre "Gris Oscuro", "Rojo Vino" y "Azul Rey" — las tres clases con más
moléculas y con rangos de afinidad contiguos.

## 3. Regresión — barrido de K (notebook 01, K=1..20)

MSE y R² sobre el conjunto de prueba (30%), reporte detallado en K=3.

| Matriz de distancia | MSE (K=3) | R² (K=3) | Mejor R² | Mejor K |
|---|---|---|---|---|
| Negative Distance | 47.81 | -13.49 | -1.38 | K=20 |
| Positive Column Distance | 6.61 | -1.00 | -0.22 | K=2 |
| Mixed 50-50 | 3.76 | -0.14 | -0.14 | K=3 |
| Mixed 30-70 | 3.49 | -0.06 | -0.06 | K=3 |
| Mixed 70-30 | 10.95 | -2.32 | -0.49 | K=10 |

El R² es negativo en absolutamente todas las combinaciones evaluadas**
(incluso en su mejor K), lo que significa que el modelo kNN de regresión, sobre esta
matriz de distancias, predice peor que simplemente devolver el promedio de afinidad del
conjunto de entrenamiento. `Mixed 30-70` es la menos mala (R²=-0.06, casi neutra), y
`Negative Distance` es por lejos la peor (probablemente por la presencia de valores
atípicos/outliers de afinidad — hay moléculas con afinidad 37, 40 y 61 frente a una mediana
mucho más baja, y el MSE en esa matriz es un orden de magnitud mayor que en las demás).

## 4. Pipeline modular con K=5 fijo (notebook 02) — contraste

| Matriz de distancia | Accuracy (K=5) | MSE (K=5) | R² (K=5) |
|---|---|---|---|
| Negative Distance | 0.290 | 29.19 | -11.11 |
| Positive Column Distance | 0.290 | 6.17 | -1.56 |
| Mixed 50-50 | 0.194 | 2.76 | -0.15 |
| Mixed 30-70 | 0.258 | 2.66 | -0.10 |
| Mixed 70-30 | 0.226 | 5.20 | -1.16 |

Consistente con el barrido de K: **el patrón se repite independientemente del método o del
valor fijo de K** — la matriz `Mixed 30-70` es sistemáticamente la más estable (mejor R²,
accuracy razonable), y `Negative Distance` es la más inestable.

## 5. Dendrogramas coloreados (patrones)

`figuras/dendrograma_*.png` — clustering jerárquico (enlace promedio) de las 153 moléculas
conocidas, coloreado por categoría de afinidad. En las 5 matrices, la gran mayoría de
ramas quedan en **gris** (clústeres con mezcla de categorías), con solo unos pocos
sub-clústeres pequeños y puros (2-3 hojas) del mismo color. Esto es consistente con la
clasificación débil: **La matriz de distancias de MEP no separa
limpiamente las categorías de afinidad** tal como están definidas actualmente.

## 6. Predicciones para las 20 moléculas de pose del GC4

Guardadas en `tablas/predicciones_clase_*.csv` (probabilidad por categoría, notebook 01,
K=3), `tablas/predicciones_afinidad_*.csv` (valor de afinidad, notebook 01, K=3), y sus
equivalentes `knc_predicciones_*_K5_*.csv` (notebook 02, K=5).

**Importante:** dado el desempeño débil de validación de la sección 2-4 (accuracy cercana
al baseline, R² negativo), estas predicciones deben tratarse como **una primera
aproximación, no como resultados confiables para tomar decisiones**. 

## Conclusión

El pipeline de kNN/kNC sobre las matrices de distancia de MEP, tal como está planteado
(bins de color fijos + kNN con distancia precomputada), **funciona correctamente a nivel
de código** pero **su poder predictivo es limitado**: la clasificación apenas iguala o supera
el baseline de clase mayoritaria, y la regresión no logra superar la media como predictor
en ninguna configuración. 
