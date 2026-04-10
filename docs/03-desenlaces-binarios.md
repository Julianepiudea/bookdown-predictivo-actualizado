# Modelado de desenlaces binarios

Los desenlaces binarios son probablemente los más frecuentes en investigación médica. Responden preguntas como: ¿presentó el evento?, ¿murió?, ¿tuvo complicación?, ¿hay enfermedad presente?, ¿recayó o no recayó?

## Regresión logística: modelo base

La regresión logística transforma la probabilidad del evento mediante la función logit:

\[
\log\left(\frac{p_i}{1-p_i}\right)=\beta_0+\beta_1 X_{1i}+\cdots+\beta_p X_{pi}
\]

donde \(p_i\) es la probabilidad del evento para el individuo \(i\).

### Interpretación

El exponente de cada coeficiente, \(e^{\beta_j}\), corresponde a un **odds ratio** ajustado. Sin embargo, en predicción interesa más la probabilidad estimada que la interpretación aislada del parámetro.

### Ventajas

- estima probabilidades entre 0 y 1;
- tiene buena interpretabilidad;
- funciona bien con tamaños muestrales moderados;
- admite términos no lineales e interacciones;
- puede regularizarse.

## Modelado de continuas dentro de la logística

Una mala práctica frecuente consiste en categorizar edad, presión arterial, biomarcadores u otras continuas. Esto introduce pérdida de información y puntos de corte artificiales.

En su lugar, es preferible:

- mantener la escala continua;
- modelar no linealidad con splines;
- centrar o estandarizar cuando sea útil para estabilidad numérica.

## Métodos de *machine learning* para clasificación

### Árboles de decisión

Generan reglas del tipo “si-entonces”. Son intuitivos, pero pueden ser inestables y propensos al sobreajuste.

### Bosques aleatorios

Promedian muchos árboles y reducen varianza. Capturan interacciones y no linealidad sin necesidad de especificarlas de antemano.

### *Boosting*

Construye secuencialmente árboles u otros modelos débiles, corrigiendo errores previos. Puede alcanzar gran capacidad predictiva, aunque con mayor complejidad de ajuste.

### SVM

Buscan hiperplanos que separen clases maximizando el margen. Pueden usar núcleos para capturar fronteras no lineales.

### Redes neuronales

Son modelos altamente flexibles con capacidad para detectar patrones complejos, pero requieren volumen de datos, regularización y procesos rigurosos de validación.

## Regularización en modelos binarios

### LASSO

Minimiza la pérdida logística añadiendo una penalización \(L_1\). Su utilidad radica en que selecciona variables al contraer algunas a cero.

\[
\min_{\beta}\left\{-\ell(\beta)+\lambda \sum_{j=1}^{p}|\beta_j|\right\}
\]

### Ridge

Penaliza mediante \(L_2\). No elimina variables, pero estabiliza coeficientes en presencia de colinealidad.

### Elastic Net

Combina ambas penalizaciones. Suele ser una opción práctica cuando hay muchos predictores correlacionados.

## Evaluación del desempeño: dimensiones clave

En desenlaces binarios, el desempeño no se resume en una sola cifra.

### Discriminación

Se refiere a la capacidad de separar individuos con y sin el evento. La medida más utilizada es el estadístico \(c\), equivalente al AUC de la curva ROC.

\[
AUC = P(\hat{p}_{evento} > \hat{p}_{no\ evento})
\]

Un AUC de 0.5 sugiere azar; valores más altos indican mejor discriminación. Sin embargo, un AUC elevado no garantiza buena calibración.

### Calibración

La calibración evalúa el acuerdo entre probabilidades predichas y frecuencias observadas. Puede resumirse mediante:

- intercepto de calibración;
- pendiente de calibración;
- gráfico de calibración.

Un modelo ideal tiene intercepto 0 y pendiente 1.

### Rendimiento global

El **Brier score** mide el error cuadrático medio entre lo observado y lo predicho:

\[
BS=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{p}_i)^2
\]

Cuanto más cercano a 0, mejor es el rendimiento global.

### Utilidad clínica

Un modelo puede discriminar y calibrar bien, pero no necesariamente ser útil en decisiones clínicas. Por ello, el análisis de curva de decisión estima el beneficio neto para distintos umbrales de decisión [@vickers2006].

## Tamaño de muestra

La vieja regla de “10 eventos por variable” fue útil como aproximación histórica, pero hoy se considera insuficiente como criterio único. Los enfoques modernos proponen tamaños de muestra basados en:

- precisión de la incidencia global;
- optimismo aceptable;
- *shrinkage* objetivo;
- pequeña diferencia entre desempeño aparente y ajustado [@riley2019].

## Aplicación práctica: modelo binario con datos de UCI

A continuación se presenta un recorrido docente usando la base `icu` de la librería `aplore3`. El ejemplo integra exploración, faltantes, preprocesamiento, separación entrenamiento/prueba y selección de variables con LASSO.

### Planteamiento del problema

Se busca predecir el estado vital al egreso hospitalario (`sta`) a partir de variables demográficas, clínicas y de laboratorio medidas al ingreso a UCI. Este tipo de problema es típicamente pronóstico.

### Variables de interés

Entre las variables útiles para el ejercicio se encuentran:

- edad (`age`);
- sexo (`gender`);
- raza (`race`);
- servicio (`ser`);
- cáncer como problema actual (`can`);
- falla renal crónica (`crn`);
- infección probable al ingreso (`inf`);
- CPR previo (`cpr`);
- presión sistólica (`sys`);
- frecuencia cardiaca (`hra`);
- tipo de admisión (`type`);
- creatinina (`cre`);
- nivel de conciencia (`loc`).

### Carga y revisión inicial


``` r
library(aplore3)
library(tidyverse)

data(icu)
str(icu)

# revisar y dejar el desenlace como factor binario
icu <- icu %>%
  mutate(
    sta = factor(sta)
  )
```

### Revisión de faltantes


``` r
sum(is.na(icu))

library(naniar)
library(VIM)

miss_var_summary(icu)

vis_miss(icu) +
  theme(axis.text.x = element_text(angle = 90))

aggr(
  icu,
  col = c("navyblue", "red"),
  numbers = TRUE,
  sortVars = TRUE,
  labels = names(icu),
  cex.axis = .7,
  gap = 3,
  ylab = c("Histograma de faltantes", "Patrón")
)
```

### Imputación simple: solo con fines pedagógicos

La imputación simple puede ilustrar la lógica básica de reemplazar valores faltantes, aunque no es la estrategia recomendada para el análisis final.


``` r
icu_simple <- icu %>%
  mutate(
    age = if_else(is.na(age), mean(age, na.rm = TRUE), age),
    sys = if_else(is.na(sys), median(sys, na.rm = TRUE), sys)
  )
```

### Imputación múltiple con MICE


``` r
library(mice)

imp_icu <- mice(icu, m = 5, method = "pmm", seed = 500)

summary(imp_icu)

icu_complete <- complete(imp_icu, 1)

stripplot(imp_icu, age ~ .imp, pch = 20, cex = 1.2)
```

### Comparación visual antes y después de imputar


``` r
ggplot() +
  geom_density(data = icu, aes(x = age), color = "blue", linewidth = 1, linetype = "dashed") +
  geom_density(data = icu_complete, aes(x = age), color = "red", linewidth = 1) +
  labs(
    title = "Distribución de edad",
    subtitle = "Azul: base original con faltantes | Rojo: base imputada",
    x = "Edad",
    y = "Densidad"
  ) +
  theme_minimal()
```

## Valores atípicos y transformación

Los valores atípicos pueden ser errores o representar pacientes verdaderamente extremos. En modelado predictivo no deben eliminarse de forma automática.

### Regla univariada de Tukey

Se considera atípico un valor fuera de:

\[
[Q_1 - 1.5 \times IQR,\; Q_3 + 1.5 \times IQR]
\]

### Distancia de Mahalanobis

Permite detectar observaciones inusuales en la combinación de varias variables, no solo en una escala aislada.


``` r
ggplot(icu, aes(y = sys)) +
  geom_boxplot(fill = "lightblue", outlier.color = "red") +
  labs(title = "Boxplot de presión sistólica", y = "mmHg") +
  theme_minimal()

icu_num <- icu %>% select(age, sys, hra)
m_dist <- mahalanobis(icu_num, colMeans(icu_num, na.rm = TRUE), cov(icu_num, use = "complete.obs"))
cutoff <- qchisq(0.999, df = ncol(icu_num))
outliers_multi <- which(m_dist > cutoff)
length(outliers_multi)
```

### Transformación logarítmica y estandarización


``` r
icu <- icu %>%
  mutate(
    log_sys = log1p(sys)
  )
```

La estandarización es particularmente útil antes de métodos penalizados como LASSO.

## Análisis descriptivo estratificado por desenlace


``` r
library(tableone)

vars <- names(icu)[!names(icu) %in% c("id", "sta")]
tabla1 <- CreateTableOne(vars = vars, strata = "sta", data = icu, test = TRUE)
print(tabla1, smd = TRUE)
```

### Gráficos exploratorios


``` r
ggplot(icu, aes(x = sta, y = age, fill = sta)) +
  geom_boxplot(outlier.shape = NA, alpha = 0.5) +
  geom_jitter(width = 0.2, alpha = 0.4, aes(color = sta)) +
  labs(title = "Edad según estado vital", x = "Estado al alta", y = "Años") +
  theme_minimal()

ggplot(icu, aes(x = sta, y = sys, fill = sta)) +
  geom_boxplot(outlier.shape = NA, alpha = 0.5) +
  geom_jitter(width = 0.2, alpha = 0.4, aes(color = sta)) +
  labs(title = "Presión sistólica según estado vital", x = "Estado al alta", y = "mmHg") +
  theme_minimal()
```


``` r
plot_cat <- function(data, var, label) {
  ggplot(data, aes(x = !!rlang::sym(var), fill = sta)) +
    geom_bar(position = "fill") +
    labs(
      title = paste("Proporción del desenlace según", label),
      y = "Proporción",
      x = label
    ) +
    scale_y_continuous(labels = scales::percent) +
    theme_minimal()
}

plot_cat(icu, "type", "Tipo de admisión")
plot_cat(icu, "inf", "Infección presente")
plot_cat(icu, "po2", "PO2")
plot_cat(icu, "pco", "PCO2")
```

## Separación entrenamiento-prueba

La división entrenamiento/prueba es útil para enseñanza, aunque no reemplaza la necesidad de validación interna rigurosa.


``` r
library(caret)
set.seed(2026)

trainIndex <- createDataPartition(icu$sta, p = 0.75, list = FALSE)

train_set <- icu[trainIndex, ]
test_set  <- icu[-trainIndex, ]

nrow(train_set)
nrow(test_set)
```

> **Nota metodológica:** si se usa un conjunto de prueba, la imputación y el preprocesamiento deben estimarse en el conjunto de entrenamiento y luego aplicarse al de prueba para evitar **fuga de información**.

## Selección de variables con LASSO

En presencia de muchos predictores y posible colinealidad, LASSO puede ayudar a reducir complejidad.


``` r
library(glmnet)

x_train <- model.matrix(sta ~ ., data = train_set)[, -1]
y_train <- ifelse(train_set$sta == "Died", 1, 0)

cv_lasso <- cv.glmnet(
  x = x_train,
  y = y_train,
  family = "binomial",
  alpha = 1
)

plot(cv_lasso)
coef(cv_lasso, s = "lambda.min")
coef(cv_lasso, s = "lambda.1se")
```

### Lectura del gráfico de validación cruzada

- el eje horizontal inferior muestra la magnitud de la penalización \(\lambda\);
- el eje superior muestra cuántos coeficientes no nulos permanecen;
- `lambda.min` produce el menor error medio de validación cruzada;
- `lambda.1se` elige un modelo más parsimonioso dentro de una banda razonable de error.

## Ajuste final y evaluación

Una estrategia común es usar las variables seleccionadas por LASSO para ajustar después un modelo logístico final y evaluar:

- AUC o estadístico \(c\);
- Brier score;
- intercepto y pendiente de calibración;
- curva de decisión si el modelo va a respaldar decisiones.

## Recomendaciones prácticas

1. Definir el momento exacto de predicción.
2. Mantener variables continuas en su escala original siempre que sea posible.
3. Revisar faltantes y evitar imputación simple como análisis final.
4. Separar claramente exploración, preprocesamiento y validación.
5. Recordar que LASSO ayuda a estabilizar el modelo, pero no reemplaza el juicio clínico ni la evaluación de desempeño.
