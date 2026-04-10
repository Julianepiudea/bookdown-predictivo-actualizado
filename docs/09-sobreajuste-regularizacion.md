# Sobreajuste, ajuste insuficiente y regularización

El sobreajuste y el ajuste insuficiente son dos expresiones de un mismo problema: elegir un nivel de complejidad inadecuado para la cantidad y calidad de información disponible.

## Sesgo, varianza y error irreducible

El error de predicción puede entenderse como la combinación de:

- **sesgo**, cuando el modelo es demasiado simple y no capta la señal;
- **varianza**, cuando el modelo es tan flexible que aprende ruido de la muestra;
- **error irreducible**, asociado a variabilidad biológica y medición imperfecta.

Formalmente, esta idea puede expresarse como:

\[
E\{y_0-\hat{f}(x_0)\}^2 = Var\{\hat{f}(x_0)\} + Bias\{\hat{f}(x_0)\}^2 + Var(\varepsilon)
\]

## Ajuste insuficiente (*underfitting*)

Ocurre cuando el modelo es demasiado rígido. Suele pasar cuando:

- se omiten predictores relevantes;
- se fuerzan relaciones lineales inadecuadas;
- se resumen fenómenos complejos en muy pocos términos;
- se usa un modelo excesivamente simple para la pregunta.

Un modelo subajustado falla tanto en entrenamiento como en nuevos datos.

## Sobreajuste (*overfitting*)

Ocurre cuando el modelo se adapta de forma excesiva a peculiaridades aleatorias de la muestra de desarrollo. Es más probable cuando:

- hay muchos predictores para pocos eventos;
- se prueban muchas transformaciones e interacciones;
- se usan procedimientos de selección inestables;
- se elige un algoritmo muy flexible sin control de complejidad.

El resultado típico es buen desempeño aparente y caída importante al validar.

## Regularización como respuesta

La regularización introduce una penalización para reducir varianza a costa de un sesgo pequeño y controlado.

### Ridge

Útil para estabilizar coeficientes cuando hay colinealidad.

### LASSO

Útil cuando además se desea seleccionar variables.

### Elastic Net

Útil cuando hay grupos de variables correlacionadas y se busca equilibrio entre selección y estabilidad.

## Parsimonia

La parsimonia no significa “el modelo más pequeño posible”, sino el modelo cuya complejidad está justificada por la información disponible y por el objetivo clínico.

## Relación con tamaño de muestra

Históricamente se usó la regla de eventos por variable. Hoy se recomienda una perspectiva más amplia, centrada en el desempeño esperado del modelo y no solo en contar parámetros [@riley2019].

## Estrategias prácticas para controlar el sobreajuste

1. planear un tamaño de muestra suficiente;
2. reducir complejidad innecesaria;
3. modelar no linealidad con métodos estables;
4. evitar selección automática ciega;
5. usar penalización cuando sea apropiado;
6. validar rigurosamente;
7. recalibrar o actualizar en poblaciones nuevas si es necesario.

## Mensaje central

En predicción clínica, un modelo ligeramente más simple pero estable suele ser preferible a uno muy complejo con desempeño frágil fuera de la muestra.
