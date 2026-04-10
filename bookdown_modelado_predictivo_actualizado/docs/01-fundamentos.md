# Fundamentos del modelado predictivo en salud

## Explicación versus predicción

Un punto de partida indispensable consiste en distinguir entre **modelos explicativos** y **modelos predictivos**. En epidemiología causal, el interés central es estimar el efecto de una exposición sobre un desenlace, controlar confusión e interpretar parámetros con lenguaje etiológico. En cambio, en predicción el objetivo es distinto: estimar con la mayor precisión posible la probabilidad de un desenlace en individuos o poblaciones específicas.

Esto tiene varias consecuencias prácticas:

- pueden incluirse predictores que no sean causales, siempre que aporten capacidad predictiva;
- la prioridad es el desempeño fuera de la muestra, no solo la significancia estadística;
- la selección de variables debe responder al momento clínico en el que se pretende usar el modelo;
- la validez del modelo depende fuertemente de la calibración, la discriminación y la generalización.

## El paradigma de Shmueli

Shmueli propuso una distinción conceptual especialmente útil para cursos de modelado: **explicar** y **predecir** no son sinónimos. Un modelo puede ser muy bueno para describir una relación causal o asociativa en la muestra observada, pero no necesariamente para anticipar bien qué ocurrirá en individuos nuevos. Del mismo modo, un modelo con excelente capacidad de predicción puede incluir variables que no sean interpretables en clave causal.

### Modelado explicativo

- busca probar hipótesis y contrastar teoría;
- privilegia interpretabilidad, control de confusión y estimación de parámetros;
- suele interesarse por razones de odds, riesgos relativos, coeficientes y valores *p*.

### Modelado predictivo

- busca minimizar error de predicción en nuevos datos;
- privilegia calibración, discriminación y generalización;
- acepta como útiles variables que ayuden a predecir, aun si no son causales.

En salud pública, confundir ambos propósitos puede ser problemático. Un modelo útil para identificar factores asociados a un desenlace no necesariamente sirve para triaje, priorización de recursos o apoyo a decisiones clínicas.

## ¿Qué es un modelo predictivo?

Un modelo predictivo es una función que combina predictores para estimar un resultado futuro o actual. Ese resultado puede ser:

- un valor continuo, como presión arterial o puntaje de dolor;
- un desenlace binario, como muerte sí/no o complicación sí/no;
- una categoría múltiple, como tipo de diagnóstico;
- un desenlace ordinal, como gravedad leve, moderada o severa;
- un tiempo hasta un evento, como recurrencia, hospitalización o muerte.

La elección del modelo depende de la estructura del desenlace y de la pregunta clínica o epidemiológica.

## Fundamentación matemática general

En términos abstractos, el problema predictivo puede representarse como:

\[
Y = f(X) + \varepsilon
\]

donde \(Y\) es el desenlace, \(X\) representa el conjunto de predictores, \(f(X)\) es la parte sistemática del fenómeno y \(\varepsilon\) corresponde al error irreducible. El objetivo del modelado es aproximar \(f\) mediante una función estimada \(\hat{f}\) que produzca predicciones útiles.

## Sesgo, varianza y error irreducible

La precisión de un método predictivo puede entenderse a partir del equilibrio entre **sesgo** y **varianza**. Para un punto \(x_0\), el error esperado puede descomponerse como:

\[
E\{y_0-\hat{f}(x_0)\}^2 = Var\{\hat{f}(x_0)\} + Bias\{\hat{f}(x_0)\}^2 + Var(\varepsilon)
\]

- **sesgo**: error por usar un modelo demasiado simple o mal especificado;
- **varianza**: inestabilidad del modelo si cambiara la muestra de entrenamiento;
- **error irreducible**: variabilidad biológica y de medición que ningún algoritmo puede eliminar del todo.

Esta descomposición ayuda a entender por qué un modelo extremadamente flexible puede verse muy bien en la muestra original y fallar en datos nuevos.

## Ciclo general de desarrollo

Un desarrollo metodológicamente sólido suele seguir estas etapas:

1. **Definición del objetivo**: diagnóstico o pronóstico; población objetivo; momento de predicción; contexto de uso.
2. **Definición del desenlace**: operacionalización clara, horizonte temporal y reglas de adjudicación.
3. **Selección de predictores candidatos**: con base en conocimiento clínico, factibilidad de medición y disponibilidad en el momento de uso.
4. **Preparación de datos**: exploración, consistencia, codificación, manejo de valores extremos y datos faltantes.
5. **Especificación del modelo**: forma funcional de predictores, términos no lineales, interacciones, regularización.
6. **Estimación y ajuste**: entrenamiento del modelo y, si aplica, ajuste de hiperparámetros.
7. **Evaluación del desempeño**: discriminación, calibración, rendimiento global y utilidad clínica.
8. **Validación**: interna y, idealmente, externa.
9. **Presentación e implementación**: fórmula, tabla de puntuación, nomograma o calculadora.

## Métodos clásicos versus *machine learning*

En salud, la comparación entre modelos clásicos y *machine learning* no debe abordarse como una competencia ideológica sino como una decisión metodológica.

### Métodos clásicos

Los métodos clásicos incluyen la regresión lineal, la regresión logística, los modelos ordinales y los modelos de Cox. Sus ventajas más relevantes son:

- transparencia y facilidad de interpretación;
- estabilidad razonable con muestras moderadas;
- buena integración con conocimiento clínico previo;
- validación e implementación relativamente sencillas.

### Métodos de *machine learning*

Los algoritmos de *machine learning* incluyen árboles, bosques aleatorios, *boosting*, redes neuronales, SVM y otros ensambles. Pueden resultar útiles cuando:

- hay interacciones complejas difíciles de preespecificar;
- se sospechan relaciones muy no lineales;
- el volumen de datos es grande;
- el objetivo principal es maximizar predicción y no interpretación causal.

Sin embargo, en salud no debe asumirse que el *machine learning* superará automáticamente a la regresión. En muestras pequeñas o moderadas, y con relación señal-ruido modesta, los modelos tradicionales continúan mostrando un desempeño muy competitivo [@steyerberg2019; @harrell2015].

## El proceso de modelado predictivo

Aunque la práctica real es iterativa, puede resumirse en cuatro grandes momentos:

### 1. Definir el objetivo

Antes de tocar los datos, hay que precisar si el problema es diagnóstico o pronóstico, cuál será la unidad de análisis y qué decisión se espera informar con el modelo.

### 2. Preparar y comprender los datos

Esto incluye revisar calidad, consistencia, faltantes, valores extremos, escalas de medición y necesidad de ingeniería de características.

### 3. Construir el modelo

Se selecciona el marco analítico adecuado según el desenlace y el contexto: regresión lineal, logística, Cox, regularización o algoritmos más flexibles.

### 4. Evaluar y validar

El desempeño aparente nunca es suficiente. Todo modelo debe someterse a validación y a una revisión explícita de su calibración, discriminación y utilidad clínica.

## Principios transversales

A lo largo de todo el libro, conviene mantener presentes cinco principios:

1. **No dicotomizar variables continuas sin una razón fuerte**.
2. **No usar casos completos por defecto cuando existen datos faltantes relevantes**.
3. **No confiar en el desempeño aparente como evidencia suficiente**.
4. **No interpretar la discriminación como sinónimo de validez clínica**.
5. **No olvidar que un modelo útil debe ser implementable y transportable**.

## Mensaje central

El modelado predictivo en salud no empieza con el algoritmo final; empieza con una definición rigurosa del problema y termina con una evaluación honesta de su utilidad real.
