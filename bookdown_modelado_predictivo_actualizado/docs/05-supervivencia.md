# Análisis de supervivencia: tiempo hasta el evento

Los modelos de supervivencia son apropiados cuando interesa no solo si ocurrió un desenlace, sino **cuándo** ocurrió. En salud, esto es esencial para recurrencia, mortalidad, falla de tratamiento, reingreso o progresión clínica.

## Conceptos básicos

### Tiempo al evento

Es el intervalo entre un origen bien definido y la ocurrencia del desenlace.

### Censura

La censura ocurre cuando no se observa el tiempo exacto del evento, aunque se sabe que hasta cierto momento no había ocurrido. Ejemplos:

- fin administrativo del estudio;
- pérdida de seguimiento;
- retiro sin evento.

## Kaplan-Meier

La curva de Kaplan-Meier estima la probabilidad de permanecer libre del evento en el tiempo. Es muy útil para descripción y visualización.

Sin embargo, tiene limitaciones:

- es principalmente descriptiva;
- no ajusta adecuadamente múltiples covariables;
- obliga a categorizar continuas si se quiere comparar grupos simples.

## Modelo de Cox

El modelo de riesgos proporcionales de Cox es el estándar en supervivencia:

\[
h(t|X)=h_0(t)\exp(\beta_1X_1+\cdots+\beta_pX_p)
\]

donde \(h_0(t)\) es el riesgo basal y los coeficientes se interpretan como razones de riesgo (*hazard ratios*).

### ¿Por qué se considera semiparamétrico?

Porque no exige una forma paramétrica específica para el riesgo basal, pero sí modela de manera paramétrica el efecto relativo de los predictores.

### Supuesto de riesgos proporcionales

El efecto de cada predictor se asume constante en el tiempo. Si se viola, pueden necesitarse interacciones con tiempo, estratificación u otros modelos.

## Modelos paramétricos

Los modelos exponencial, Weibull y otros imponen una forma funcional al riesgo basal. Son útiles cuando interesa:

- extrapolar más allá del seguimiento observado;
- estimar funciones de supervivencia completas;
- obtener predicciones absolutas a largo plazo con una estructura más cerrada.

## Supervivencia y *machine learning*

Entre las extensiones más usadas se encuentran:

- bosques aleatorios para supervivencia;
- *boosting* para Cox;
- LASSO para selección de variables en Cox;
- redes neuronales adaptadas a tiempo al evento.

Estos métodos pueden capturar mayor complejidad, aunque la validación sigue siendo indispensable.

## Evaluación del desempeño en supervivencia

### Discriminación

Se usa con frecuencia el índice de concordancia o estadístico \(c\), que resume la proporción de pares comparables en los que el modelo ordena correctamente los riesgos.

### Calibración

Debe evaluarse en tiempos específicos, por ejemplo a 1, 3 o 5 años, comparando riesgo predicho versus riesgo observado.

### Brier score dependiente del tiempo

En supervivencia, el Brier score se calcula en tiempos fijos y requiere métodos que manejen la censura, como ponderación por probabilidad inversa de censura.

## Aspectos críticos en investigación

### Definir correctamente el tiempo cero

El origen temporal debe ser coherente con la pregunta de predicción. Un error en este punto induce sesgos serios.

### No confundir incidencia acumulada con hazard

El *hazard ratio* no es lo mismo que una razón de riesgos acumulados. La interpretación debe respetar la escala del modelo.

### Manejo de continuas

Al igual que en binario, no conviene dicotomizar edad, laboratorio o biomarcadores. Los splines son una estrategia preferible.

## Aplicación práctica: modelo de supervivencia con la base PBC

En esta sección se utiliza la base `pbc` de la librería `survival`, clásica en docencia para análisis de supervivencia. El ejemplo integra pérdida MCAR simulada, imputación, análisis descriptivo, Kaplan-Meier, Nelson-Aalen, LASSO-Cox, nomograma, curva de decisión y una calculadora.

## Fundamento del modelo de Cox

A diferencia de la logística, aquí no predecimos solamente “sí o no”, sino el riesgo de que el evento ocurra a lo largo del tiempo:

\[
h(t | X) = h_0(t) \exp(\beta_1 X_1 + \beta_2 X_2 + \cdots + \beta_k X_k)
\]

donde:

- \(h(t|X)\): hazard o riesgo instantáneo en el tiempo \(t\);
- \(h_0(t)\): riesgo basal;
- \(\exp(\beta)\): hazard ratio asociado a un predictor.

## Generación de pérdida MCAR e imputación

En ejercicios docentes suele ser útil introducir faltantes artificiales para practicar imputación. En supervivencia, la imputación debe tener en cuenta el desenlace y, con frecuencia, el tiempo de seguimiento o el estimador de Nelson-Aalen.


``` r
library(survival)
library(mice)
library(naniar)

data(pbc)

vars_imp <- c("time", "status", "age", "sex", "bili", "albumin", "ast", "chol", "copper")
pbc_sub <- pbc[, vars_imp]

set.seed(42)
pbc_miss <- pbc_sub

for (j in seq_along(pbc_miss)) {
  idx <- sample(seq_len(nrow(pbc_miss)), size = round(0.12 * nrow(pbc_miss)))
  pbc_miss[idx, j] <- NA
}

vis_miss(pbc_miss)

imp <- mice(pbc_miss, m = 5, method = "rf", maxit = 5, seed = 500)
pbc_imp <- complete(imp)

pbc_final <- pbc
pbc_final[, vars_imp] <- pbc_imp
```

> **Nota docente:** en análisis reales conviene justificar con claridad el mecanismo de pérdida y elegir un modelo de imputación compatible con la estructura del dato de supervivencia.

## Análisis descriptivo general

Antes de ajustar un modelo de Cox, conviene describir estadísticamente la cohorte y explorar la distribución de las variables continuas y categóricas.


``` r
library(ggplot2)
library(tidyr)

pbc_final %>%
  select(where(is.numeric), -id, -trt, -stage, -status, -hepato, -edema, -spiders, -ascites) %>%
  pivot_longer(everything(), names_to = "variable", values_to = "valor") %>%
  ggplot(aes(x = valor)) +
  geom_histogram(fill = "steelblue", color = "white", bins = 30) +
  facet_wrap(~ variable, scales = "free") +
  theme_classic()
```


``` r
library(kableExtra)

pbc_final %>%
  select(where(is.numeric), -id, -trt) %>%
  pivot_longer(everything(), names_to = "variable", values_to = "valor") %>%
  group_by(variable) %>%
  summarise(
    n        = sum(!is.na(valor)),
    media    = round(mean(valor, na.rm = TRUE), 2),
    ds       = round(sd(valor, na.rm = TRUE), 2),
    mediana  = round(median(valor, na.rm = TRUE), 2),
    q1       = round(quantile(valor, 0.25, na.rm = TRUE), 2),
    q3       = round(quantile(valor, 0.75, na.rm = TRUE), 2),
    min      = round(min(valor, na.rm = TRUE), 2),
    max      = round(max(valor, na.rm = TRUE), 2),
    perdidos = sum(is.na(valor))
  ) %>%
  kable(caption = "Medidas de resumen de variables cuantitativas") %>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed"))
```


``` r
vars_cuali <- c("trt", "sex", "ascites", "hepato", "spiders", "edema", "stage")

pbc_final %>%
  select(all_of(vars_cuali)) %>%
  mutate(
    trt     = factor(trt, levels = c(1, 2), labels = c("D-penicilamina", "Placebo")),
    sex     = factor(sex, levels = c("m", "f"), labels = c("Masculino", "Femenino")),
    ascites = factor(ascites, levels = c(0, 1), labels = c("No", "Sí")),
    hepato  = factor(hepato, levels = c(0, 1), labels = c("No", "Sí")),
    spiders = factor(spiders, levels = c(0, 1), labels = c("No", "Sí")),
    edema   = factor(edema, levels = c(0, 0.5, 1),
                     labels = c("No edema", "Tratamiento exitoso/No tratado", "Edema a pesar de diuréticos")),
    stage   = factor(stage, levels = c(1, 2, 3, 4),
                     labels = c("Etapa I", "Etapa II", "Etapa III", "Etapa IV"))
  ) %>%
  pivot_longer(everything(), names_to = "variable", values_to = "categoria") %>%
  group_by(variable, categoria) %>%
  summarise(n = n(), .groups = "drop") %>%
  group_by(variable) %>%
  mutate(
    porcentaje = round(n / sum(n) * 100, 1),
    resultado  = paste0(n, " (", porcentaje, "%)")
  ) %>%
  mutate(variable = ifelse(duplicated(variable), "", variable)) %>%
  select(variable, categoria, resultado)
```

## Recodificación del desenlace

En `pbc`, la variable `status` original tiene tres estados. Para un modelo simple de supervivencia del tipo evento/no evento, a veces se recategoriza así:

- 0: censura o trasplante;
- 1: muerte.


``` r
pbc_final <- pbc_final %>%
  mutate(
    status = case_when(
      status %in% c(0, 1) ~ 0,
      status == 2 ~ 1,
      TRUE ~ NA_real_
    )
  )
```

## Función de supervivencia de Kaplan-Meier


``` r
library(ggsurvfit)

kmfit <- survfit(Surv(time, status) ~ 1, data = pbc_final)

survfit2(Surv(time, status) ~ 1, data = pbc_final) %>%
  ggsurvfit() +
  add_confidence_interval(fill = "steelblue", alpha = 0.2) +
  add_risktable() +
  add_quantile(y_value = 0.5, color = "red", linewidth = 0.75) +
  scale_x_continuous(breaks = seq(0, 5000, by = 500)) +
  scale_y_continuous(limits = c(0, 1), labels = scales::percent) +
  labs(
    title = "Curva de Kaplan-Meier",
    x = "Tiempo (días)",
    y = "Probabilidad de supervivencia"
  ) +
  theme_classic()
```

La línea horizontal de 0.5 y su intersección con la curva permiten visualizar la **mediana de supervivencia**, es decir, el tiempo en el que la probabilidad de seguir vivo cae a 50%.

## Función de riesgo acumulado de Nelson-Aalen


``` r
survfit2(Surv(time, status) ~ 1, data = pbc_final) %>%
  ggsurvfit(type = "cumhaz") +
  add_confidence_interval(fill = "steelblue", alpha = 0.2) +
  add_risktable() +
  scale_x_continuous(breaks = seq(0, 5000, by = 500)) +
  labs(
    title = "Riesgo acumulado de Nelson-Aalen",
    x = "Tiempo (días)",
    y = "Riesgo acumulado H(t)"
  ) +
  theme_classic()
```

## Comparación de curvas por tratamiento


``` r
survfit2(Surv(time, status) ~ trt, data = pbc_final) %>%
  ggsurvfit() +
  add_confidence_interval(alpha = 0.2) +
  add_risktable() +
  add_quantile(y_value = 0.5, color = "red", linewidth = 0.75) +
  scale_x_continuous(breaks = seq(0, 5000, by = 500)) +
  scale_y_continuous(limits = c(0, 1), labels = scales::percent) +
  labs(
    title = "Kaplan-Meier por tratamiento",
    x = "Tiempo (días)",
    y = "Probabilidad de supervivencia"
  ) +
  theme_classic()
```

### Pruebas de comparación


``` r
survdiff(Surv(time, status) ~ trt, data = pbc_final)          # Log-rank
survdiff(Surv(time, status) ~ trt, data = pbc_final, rho = 1)  # Breslow
survdiff(Surv(time, status) ~ trt, data = pbc_final, rho = 0.5) # Tarone-Ware
```

| Situación | Prueba orientativa |
|:--|:--|
| Diferencias relativamente constantes en el tiempo | Log-rank |
| Diferencias tempranas | Breslow |
| No está claro en qué momento difieren las curvas | Tarone-Ware |
| Muchas censuras y atención a periodos iniciales | Peto-Peto o variantes relacionadas |

## Selección de variables con LASSO-Cox

En supervivencia, la función objetivo cambia: ya no se minimizan residuos cuadrados, sino que se maximiza una verosimilitud parcial penalizada.

\[
\log L(\beta) - \lambda \sum_{j=1}^{p} |\beta_j|
\]


``` r
library(glmnet)

pbc_lasso <- pbc_final[!is.na(pbc_final$time) & !is.na(pbc_final$status), ]

x_lasso <- as.matrix(pbc_lasso[, c("age", "bili", "albumin", "ast", "chol")])
x_lasso <- cbind(x_lasso, sex = ifelse(pbc_lasso$sex == "f", 1, 0))

y_lasso <- Surv(pbc_lasso$time, pbc_lasso$status)

set.seed(123)
cv_lasso_cox <- cv.glmnet(x_lasso, y_lasso, family = "cox", alpha = 1)

plot(cv_lasso_cox)
coef(cv_lasso_cox, s = "lambda.min")
coef(cv_lasso_cox, s = "lambda.1se")
```

## Modelo de Cox final y nomograma


``` r
library(rms)

dd <- datadist(pbc_lasso)
options(datadist = "dd")

f_final <- cph(
  Surv(time, status) ~ age + bili + albumin + ast + chol + sex,
  data = pbc_lasso,
  x = TRUE, y = TRUE, surv = TRUE
)

S <- Survival(f_final)

nomo <- nomogram(
  f_final,
  fun = list(
    function(x) S(1095, x),
    function(x) S(1825, x)
  ),
  funlabel = c("Prob. supervivencia 3 años", "Prob. supervivencia 5 años")
)

plot(nomo, xfrac = 0.3, cex.axis = 0.8)
```

El nomograma traduce el modelo en una herramienta gráfica de puntajes que permite calcular probabilidades individuales sin exponer al usuario a la ecuación completa.

## Curva de decisión


``` r
library(dcurves)

pred_riesgo <- 1 - S(1825, f_final$linear.predictors)
pbc_lasso$riesgo_5a <- as.numeric(pred_riesgo)

dca_obj <- dca(
  Surv(time, status) ~ riesgo_5a,
  data = pbc_lasso,
  time = 1825,
  label = list(riesgo_5a = "Modelo predictivo")
)

plot(dca_obj) +
  theme_minimal() +
  labs(
    title = "Curva de decisión a 5 años",
    x = "Umbral de riesgo",
    y = "Beneficio neto"
  )
```

La curva de decisión ayuda a responder una pregunta distinta a la del AUC o la calibración: **¿vale la pena usar el modelo para tomar decisiones?**

## Calculadora tipo Shiny


``` r
library(shiny)
library(bslib)

S_func <- Survival(f_final)

ui <- fluidPage(
  theme = bs_theme(bootswatch = "flatly"),
  titlePanel("Calculadora de riesgo: cirrosis biliar primaria"),
  sidebarLayout(
    sidebarPanel(
      sliderInput("age", "Edad:", 20, 80, 50),
      numericInput("bili", "Bilirrubina:", 1.5, min = 0.1, max = 30, step = 0.1),
      numericInput("albumin", "Albúmina:", 3.5, min = 1, max = 5, step = 0.1),
      numericInput("ast", "AST:", 120, min = 10, max = 500),
      numericInput("chol", "Colesterol:", 250, min = 100, max = 800),
      selectInput("sex", "Sexo:", choices = c("Femenino" = "f", "Masculino" = "m")),
      actionButton("calc", "Calcular")
    ),
    mainPanel(
      uiOutput("res_text"),
      plotOutput("surv_plot")
    )
  )
)

server <- function(input, output) {
  observeEvent(input$calc, {
    nuevo <- data.frame(
      age = input$age,
      bili = input$bili,
      albumin = input$albumin,
      ast = input$ast,
      chol = input$chol,
      sex = input$sex
    )

    lp <- predict(f_final, newdata = nuevo)
    prob_5a <- S_func(1825, lp)

    output$res_text <- renderUI({
      tagList(
        h4("Probabilidad de supervivencia a 5 años"),
        h2(paste0(round(prob_5a * 100, 1), "%"))
      )
    })

    output$surv_plot <- renderPlot({
      surv_indiv <- survfit(f_final, newdata = nuevo)
      plot(
        surv_indiv,
        conf.int = FALSE,
        lwd = 2,
        col = "royalblue",
        xlab = "Días",
        ylab = "Probabilidad de supervivencia",
        main = "Curva individualizada"
      )
      abline(v = 1825, col = "red", lty = 2)
    })
  })
}

shinyApp(ui = ui, server = server)
```

## Recomendaciones

1. Definir claramente el origen temporal.
2. Explorar censura y pérdidas de seguimiento.
3. Revisar proporcionalidad de riesgos.
4. Reportar discriminación y calibración en tiempos clínicamente relevantes.
5. Considerar nomogramas, calculadoras o aplicaciones web solo después de validar adecuadamente el modelo.
