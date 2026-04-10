---
title: "Modelado predictivo en salud"
author: "Yury Silva-Julián Galvis"
date: "09/04/2026"
site: bookdown::bookdown_site
documentclass: book
bibliography: references.bib
link-citations: yes
lang: es
description: "Bookdown sobre desarrollo, validación e implementación de modelos predictivos en salud."
---



# Presentación {-}

Este libro reúne, reorganiza y amplía el material base sobre desarrollo de modelos predictivos en salud, con énfasis en epidemiología, investigación clínica y salud pública. El objetivo no es solo describir algoritmos, sino ordenar el proceso completo de construcción de un modelo: formulación del problema, definición del desenlace, preprocesamiento, manejo de datos faltantes, especificación, validación, evaluación del desempeño, utilidad clínica e implementación.

En el campo de la predicción clínica, un modelo útil no es simplemente el que alcanza la mejor métrica aparente en la muestra de desarrollo. Un buen modelo debe ser clínicamente plausible, metodológicamente sólido, bien calibrado, estable frente a nuevas muestras y, sobre todo, útil para la toma de decisiones. Por eso, este texto sigue una lógica cercana a los marcos de trabajo propuestos para modelos de predicción clínica y para su reporte transparente [@steyerberg2019; @harrell2015; @tripod2015].

# Propósito del libro

Este bookdown fue concebido como un material de estudio y consulta para:

- estudiantes de epidemiología, bioestadística e investigación clínica;
- profesionales de salud pública que necesitan comprender la lógica del modelado predictivo;
- investigadores que buscan una guía estructurada para construir, validar e interpretar modelos;
- docentes que requieren un texto base para cursos o seminarios.

# Cómo está organizado

El contenido sigue una secuencia metodológica:

1. fundamentos generales del modelado predictivo;
2. tipos de desenlaces y modelos según la naturaleza del resultado;
3. datos faltantes, imputación y preprocesamiento;
4. métricas de desempeño, validación y utilidad clínica;
5. sobreajuste, regularización y generalización;
6. escenarios avanzados, como riesgos competitivos y predicciones dinámicas;
7. aplicaciones prácticas en desenlaces binarios y supervivencia.

# Recomendación de lectura

Aunque los capítulos pueden consultarse por separado, se recomienda leerlos en orden. La razón es que muchos errores en modelado predictivo no provienen del algoritmo final, sino de decisiones previas mal resueltas: definición inadecuada del desenlace, categorización innecesaria de variables continuas, uso incorrecto de casos completos, interpretación parcial del AUC, fuga de información entre entrenamiento y prueba, o ausencia de validación rigurosa.

# Qué incorpora esta versión

Esta versión integra dos ejes aplicados adicionales:

- un ejemplo de **modelo predictivo binario** con datos de UCI, que recorre exploración, faltantes, imputación, separación entrenamiento/prueba, LASSO y ajuste final;
- un ejemplo de **modelo predictivo de supervivencia** con la base **pbc**, que incluye Kaplan-Meier, Nelson-Aalen, comparación de curvas, LASSO-Cox, nomograma, curva de decisión y una calculadora tipo *Shiny*.

# Paquetes útiles en R
install.packages(c(
  "bookdown", "rmarkdown", "knitr", "tidyverse", "survival",
  "rms", "mice", "glmnet", "nnet", "ordinal", "riskRegression",
  "pec", "timeROC", "randomForestSRC", "cmprsk", "JMbayes2",
  "naniar", "VIM", "tableone", "ggsurvfit", "survminer",
  "kableExtra", "dcurves", "shiny", "bslib", "caret", "aplore3"
))





# Alcance

Este material prioriza el razonamiento metodológico. Cuando se mencionan algoritmos de *machine learning*, se hace desde una perspectiva aplicada a salud: cuándo pueden aportar valor, qué limitaciones tienen y por qué, en muchos contextos clínicos con tamaños muestrales moderados, los modelos de regresión siguen siendo altamente competitivos.


