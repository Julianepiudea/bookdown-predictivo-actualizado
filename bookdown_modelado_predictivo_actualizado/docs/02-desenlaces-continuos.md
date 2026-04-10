# Modelado de desenlaces continuos

Los desenlaces continuos son frecuentes en investigación clínica y epidemiológica. Ejemplos típicos incluyen presión arterial, días de estancia hospitalaria, costo de atención, biomarcadores, escalas de calidad de vida o puntajes sintomáticos.

## Regresión lineal como punto de partida

La regresión lineal ordinaria modela un desenlace continuo \(Y\) como una combinación de predictores:

\[
Y_i = \beta_0 + \beta_1 X_{1i} + \cdots + \beta_p X_{pi} + \varepsilon_i
\]

Su atractivo principal es la interpretabilidad. Cada coeficiente representa el cambio promedio esperado en el desenlace por una unidad de cambio en el predictor, manteniendo constantes los demás.

## Supuestos que deben revisarse

Aunque la regresión lineal es flexible, no debe asumirse que siempre es adecuada sin diagnóstico. Entre los aspectos clave están:

- linealidad aproximada entre predictores y desenlace;
- varianza aproximadamente constante de los residuos;
- independencia de observaciones;
- influencia de valores extremos;
- plausibilidad de normalidad de residuos cuando interesa inferencia clásica.

En contexto predictivo, el foco no es “cumplir supuestos” de forma ritual, sino garantizar que la estructura del modelo capture razonablemente el patrón de los datos y generalice bien.

## Más allá de la linealidad estricta

Muchas relaciones en salud no son rectas. La asociación entre edad, creatinina, glucosa o saturación de oxígeno y un desenlace continuo puede ser curvilínea.

### Estrategias útiles

- **Polinomios**: incorporan términos cuadráticos o cúbicos, aunque pueden ser inestables en extremos.
- **Splines cúbicos restringidos**: permiten curvas flexibles manteniendo linealidad en las colas.
- **Modelos aditivos generalizados**: reemplazan la suma lineal por funciones suaves de cada predictor.

En la práctica, los splines suelen ser una alternativa muy sólida porque combinan flexibilidad e interpretabilidad.

## Regularización en desenlaces continuos

Cuando hay muchos predictores o colinealidad importante, la estimación clásica puede volverse inestable. En esos escenarios, las técnicas de penalización reducen varianza y mejoran generalización.

### Ridge

Añade una penalización \(L_2\), es decir, sobre la suma de coeficientes al cuadrado. Reduce magnitudes, pero no elimina variables.

### LASSO

Añade una penalización \(L_1\), basada en la suma de valores absolutos. Puede llevar coeficientes exactamente a cero, por lo que actúa también como mecanismo de selección.

### Elastic Net

Combina penalización \(L_1\) y \(L_2\), siendo especialmente útil cuando existen grupos de predictores correlacionados.

## Métodos no lineales de *machine learning*

Para relaciones complejas pueden usarse:

- redes neuronales;
- *support vector regression*;
- MARS;
- bosques aleatorios;
- *boosting*.

No obstante, antes de adoptar un algoritmo complejo conviene preguntarse si el tamaño muestral, la calidad de medición y el propósito clínico justifican esa complejidad.

## Evaluación del desempeño

### Error cuadrático medio y su raíz

\[
MSE = \frac{1}{n}\sum_{i=1}^{n}(Y_i - \hat{Y}_i)^2
\]

\[
RMSE = \sqrt{MSE}
\]

El RMSE se interpreta en las mismas unidades del desenlace y penaliza con mayor fuerza los errores grandes.

### Error absoluto medio

\[
MAE = \frac{1}{n}\sum_{i=1}^{n}|Y_i - \hat{Y}_i|
\]

Suele ser más robusto frente a valores extremos que el RMSE.

### Coeficiente de determinación

\[
R^2 = 1 - \frac{\sum (Y_i - \hat{Y}_i)^2}{\sum (Y_i - \bar{Y})^2}
\]

Indica la proporción de variabilidad explicada por el modelo, aunque no sustituye la revisión de calibración ni de error absoluto.

### Calibración

En desenlaces continuos, la calibración puede revisarse comparando observados versus predichos. Un modelo ideal se alinea con una línea de 45 grados.

## Aspectos prácticos en salud

### No dicotomizar desenlaces continuos

Convertir un desenlace continuo a binario implica pérdida de información, poder estadístico y precisión. En la mayoría de los casos, es preferible modelar la variable en su escala original.

### Transformaciones del desenlace

En variables con fuerte asimetría, como costos o días de estancia, una transformación logarítmica puede ser útil. Sin embargo, debe justificarse en función del problema y de la interpretación posterior.

### Tamaño de muestra

Para desenlaces continuos no basta con pensar en “número de predictores”. También importa la varianza residual, el grado esperado de sobreajuste y la precisión requerida para las predicciones. El enfoque moderno recomienda planear tamaños de muestra que garanticen un nivel aceptable de *shrinkage* y optimismo bajo [@riley2019].

## Recomendaciones

1. Empezar por un modelo lineal bien especificado.
2. Evaluar no linealidad antes de categorizar continuas.
3. Considerar regularización cuando haya muchos predictores.
4. Reportar RMSE, MAE, \(R^2\) y calibración.
5. Validar el desempeño más allá de la muestra de desarrollo.
