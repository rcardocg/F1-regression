# ¿Cómo puedo determinar el resultado de una carrera de Formula 1?

Proyecto de regresión lineal sobre resultados de carreras de F1 (dataset "Formula 1 World Championship 1950-2024" de Kaggle).

---

# Revisión de la justificación y conclusiones del informe

> Las secciones "Métricas", "Justificación" y "Conclusiones" del PDF original contienen afirmaciones que **no se corresponden con lo que realmente se ejecutó en el notebook**. A continuación se indica, para cada una, qué decía el texto original (ANTES), por qué es incorrecto, y el texto propuesto en su lugar (DESPUÉS).

## Contexto: qué hizo realmente el notebook

El notebook entrenó **solo dos modelos**:

1. **Modelo simple** = `grid` + `total_pit_time` (2 variables predictoras; 3 columnas con la constante). No incluye `count_pit_stops`.
2. **Modelo complejo** = `grid` + `total_pit_time` + 10 variables dummy (Red Bull, Ferrari, Mercedes, Sauber; Verstappen, Russell, Norris, Piastri, Magnussen, Sargeant) = 12 predictoras (13 columnas con la constante).

`count_pit_stops` **solo se usó en el análisis de p-values (sección 6.2), nunca en ningún modelo entrenado.** Tampoco se calculó nunca la correlación entre `total_pit_time` y `count_pit_stops`.

| Métrica (test) | Modelo simple (grid + total_pit_time) | Modelo complejo (+ 10 dummies) |
|---|---|---|
| R² | 0.356 | 0.401 |
| R² ajustado (train) | 0.361 | 0.414 |
| MAE | 3.921 | 3.740 |
| RMSE | 4.943 | 4.766 |
| Accuracy* | 0.830 | 0.837 |
| Precisión** | 0.597 | 0.634 |

*Accuracy definida en el notebook como `1 − MAE/rango` (no es exactitud de clasificación).
**Precisión definida como coeficiente de correlación entre real y predicho (no es precisión de clasificación). Estas definiciones no son estándar y no deben interpretarse como tales.

## Métricas

**ANTES (incorrecto):** El PDF afirma que se evaluaron *cuatro* modelos (2, 3, 12 y 13 variables) comparando la presencia de `count_pit_stops`, citando un R² ajustado de **0.426** para el modelo de 3 variables y de **0.426/0.427** para el de 13, con MAE 3.81/3.63, RMSE 4.83/4.63, precisión 0.65 y accuracy 0.84. También afirma que `total_pit_time` y `count_pit_stops` están "altamente correlacionadas" y que eliminar cualquiera de ellas no cambia las métricas.

**Por qué es incorrecto:** No existieron tales modelos ni cifras. Ningún modelo incluyó `count_pit_stops`; los R² de 0.426/0.427 no aparecen en ningún output del notebook. La supuesta correlación/redundancia entre las dos variables de pits nunca fue calculada ni probada.

**DESPUÉS (propuesto):** Se entrenaron dos modelos y se evaluaron en train/val/test (ver tabla). El modelo complejo supera al simple en todas las métricas: R² de 0.401 frente a 0.356 en test, MAE de 3.740 frente a 3.921 y RMSE de 4.766 frente a 4.943, con valores estables entre validación y prueba (R² ~0.37-0.40), lo que indica ausencia de sobreajuste importante. La afirmación de que `total_pit_time` y `count_pit_stops` son redundantes **queda sin sustento**: no se midió su correlación ni se comparó un modelo con una u otra. Si se desea sostenerla, debe ejecutarse un modelo de 3 variables con `count_pit_stops` y verificarlo (p. ej. con VIF).

## Justificación

**ANTES (incorrecto):** "La elección de los modelos de 3 y 13 variables se justifica porque incluyen `count_pit_stops` y presentan mejor desempeño; las variables `total_pit_time` y `count_pit_stops` aportan información similar y es clave incluir al menos una."

**Por qué es incorrecto:** La premisa central es falsa: el modelo de 3 variables del notebook (que en realidad son `grid` + `total_pit_time`) **no incluye** `count_pit_stops`, y no existe comparación "con vs. sin" dicha variable. La justificación se basa en resultados que no se obtuvieron.

**DESPUÉS (propuesto):**
- El modelo simple se justifica por las variables de mayor significancia estadística e interpretabilidad: `grid` (p ≈ 3.2×10⁻⁹⁰, la más influyente, coherente con que la clasificación predice fuertemente el resultado) y `total_pit_time` (p ≈ 4.9×10⁻²¹), ambas con efecto significativo en la posición final.
- El modelo complejo añade las dummy de pilotos y escuderías que resultaron significativas (p < 0.05) **y** activas en las temporadas recientes (≥ 2024): Red Bull, Ferrari, Mercedes, Sauber y los pilotos Verstappen, Russell, Norris, Piastri, Magnussen y Sargeant.
- `count_pit_stops` fue significativo en el cribado de p-values (p ≈ 3.4×10⁻⁴⁷), pero no se incorporó a ningún modelo entrenado; **no hay evidencia en el notebook** de que sea redundante con `total_pit_time`.
- Salvedad interpretativa: el coeficiente de `total_pit_time` es **negativo** (−0.026/−0.025), es decir, a mayor tiempo en pits, mejor (menor) posición, lo que es contraintuitivo y sugiere confusión o selección (p. ej. pilotos que abandonan no vuelven a pits, o paradas asociadas a estrategias/ritmo de los equipos punteros). Debe discutirse con cautela antes de atribuirle un efecto causal.

## Conclusiones

**ANTES (incorrecto):** "El modelo de 13 variables mejora al de 3 variables, pero no lo suficiente como para justificar la complejidad; el modelo simple de 3 variables captura eficientemente la relación. `total_pit_time` y `count_pit_stops` describen el mismo fenómeno y eliminar cualquiera no modifica los resultados."

**Por qué es incorrecto:** El "modelo de 3 variables con `count_pit_stops`" no se ejecutó, por lo que la comparación y la conclusión de redundancia son insostenibles.

**DESPUÉS (propuesto):**
- El modelo complejo (12 predictoras) sí supera de forma real, aunque modesta, al simple (2 predictoras): R² 0.401 frente a 0.356 en test (+12% aprox.), MAE 3.740 frente a 3.921 (−5%) y RMSE 4.766 frente a 4.943 (−4%). La ganancia existe pero es limitada.
- `grid` es el factor dominante: por sí solo explica la mayor parte de la capacidad predictiva, en línea con la realidad del deporte (la posición de salida es el mejor indicador previo a la carrera).
- El R² ≈ 0.40 implica que más del 60% de la variabilidad queda sin explicar. Faltan factores relevantes (clima, incidentes, estrategia, fiabilidad, forma del piloto/equipo), por lo que el modelo identifica **indicadores** del resultado, pero no permite predecirlo con precisión.
- La afirmación sobre la redundancia de `total_pit_time` y `count_pit_stops` **no está demostrada** en el trabajo realizado: nunca se calculó su correlación ni se compararon modelos que las incluyeran por separado. La elección de `total_pit_time` frente a `count_pit_stops` debe basarse en un análisis adicional, no en una correlación que no se midió.
- Recomendación: incorporar datos de calificación/clasificación, rachas de forma, clima y estado mecánico, y verificar explícitamente el aporte de las variables de pits (p. ej. con un modelo de 3 variables y VIF) antes de afirmar su redundancia.

## Nota sobre el notebook

El "Modelo Simple (3 variables)" del notebook en realidad usa 2 predictoras (`grid`, `total_pit_time`) más la constante; el "Modelo complejo (13 variables)" usa 12 predictoras más la constante. Los 19 features de ingeniería (p. ej. `avg_pit_time`, `pit_time_log1p`, `grid_x_pits`, `pit_stops_per_10laps`) se construyeron pero **ninguno se usó en los modelos finales**; fueron útiles solo como etapa exploratoria del cribado de variables. Además, el PDF incluye una errata: "transformación logarítmoca" → "logarítmica".
