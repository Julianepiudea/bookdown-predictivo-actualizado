# Métricas de desempeño, validación e implementación

El desempeño de un modelo predictivo debe evaluarse en varias dimensiones. Limitarse a una sola métrica conduce a interpretaciones parciales y, a veces, engañosas.

## Dimensiones del desempeño

### Discriminación

Capacidad del modelo para asignar mayor riesgo a quienes realmente presentan el evento que a quienes no lo presentan.

### Calibración

Grado de acuerdo entre riesgo predicho y riesgo observado.

### Rendimiento global

Resume error total o cantidad de información explicada.

### Utilidad clínica

Evalúa si usar el modelo mejora decisiones respecto a estrategias por defecto.

## Métricas principales

### Para desenlaces binarios

| Dimensión | Métrica | Idea general |
|:--|:--|:--|
| Discriminación | AUC / estadístico \(c\) | Ordena correctamente pares evento/no evento |
| Calibración | intercepto, pendiente, gráfico | Compara riesgo predicho con observado |
| Rendimiento global | Brier score | Error cuadrático medio de probabilidades |
| Utilidad clínica | beneficio neto | Balance entre verdaderos positivos y falsos positivos |

### Para desenlaces continuos

| Dimensión | Métrica | Idea general |
|:--|:--|:--|
| Error | RMSE | Penaliza errores grandes |
| Error | MAE | Error absoluto promedio |
| Rendimiento global | \(R^2\) | Variación explicada |
| Calibración | observados vs predichos | Revisión visual y pendiente |

### Para supervivencia

| Dimensión | Métrica | Idea general |
|:--|:--|:--|
| Discriminación | índice de concordancia | Orden correcto de tiempos/riesgos |
| Calibración | riesgo observado vs predicho en \(t\) | Evaluación en tiempos fijos |
| Rendimiento global | Brier dependiente del tiempo | Error con manejo de censura |
| Utilidad clínica | DCA adaptada a tiempo | Beneficio neto en horizontes temporales |

## Métricas de clasificación dependientes del punto de corte

Cuando el modelo se usa para clasificar en “alto” o “bajo” riesgo según un umbral, aparecen métricas adicionales:

| Métrica | Fórmula | Cuándo es útil |
|:--|:--|:--|
| Accuracy | \((TP+TN)/(TP+TN+FP+FN)\) | Útil si las clases están balanceadas |
| Sensibilidad | \(TP/(TP+FN)\) | Cuando importa no perder casos |
| Especificidad | \(TN/(TN+FP)\) | Cuando importa evitar falsos positivos |
| Precisión (PPV) | \(TP/(TP+FP)\) | Cuando un falso positivo tiene alto costo |
| F1-score | \(2 \cdot \frac{Precisión \cdot Sensibilidad}{Precisión + Sensibilidad}\) | Balance entre precisión y sensibilidad |
| Kappa | acuerdo observado ajustado por azar | Útil cuando hay fuerte desbalance |

## Brier score

Para desenlace binario:

\[
BS=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{p}_i)^2
\]

Mide simultáneamente aspectos de calibración y discriminación. Su interpretación depende de la prevalencia, por lo que en ocasiones se usa una versión escalada.

## Estadístico \(c\)

El estadístico \(c\) es la medida estándar de discriminación. Para desenlaces binarios coincide con el AUC. Se interpreta como la probabilidad de que, al escoger un par de personas, una con el evento y otra sin él, el modelo asigne mayor riesgo a quien realmente lo presentó.

## Validación interna

La validación interna estima qué tanto optimismo tiene el desempeño aparente.

### División entrenamiento-prueba

Es simple, pero ineficiente porque sacrifica muestra de desarrollo.

### Validación cruzada

Reutiliza los datos mediante particiones repetidas. Es útil en ajuste de hiperparámetros y comparación de modelos.

### *Bootstrap*

Es una de las estrategias preferidas para predicción clínica porque permite estimar el optimismo del proceso completo de modelado, incluyendo selección de variables y ajuste [@harrell2015].

## Validación externa

Es la prueba más exigente de generalización. Puede ser:

- geográfica;
- temporal;
- por dominio asistencial;
- por población distinta.

Un modelo que funciona solo en la cohorte original aún no está listo para adopción amplia.

## Utilidad clínica y curva de decisión

El análisis de curva de decisión cuantifica el **beneficio neto** de usar el modelo para un rango de umbrales de intervención.

\[
Net\ Benefit = \frac{TP}{N} - \frac{FP}{N}\left(\frac{p_t}{1-p_t}\right)
\]

donde \(p_t\) es el umbral de riesgo que activaría una intervención.

## Presentación e implementación

Una vez validado, el modelo puede implementarse como:

- ecuación de regresión;
- tabla de puntos;
- nomograma;
- calculadora web;
- aplicación integrada al sistema clínico.

La forma de presentación debe mantener coherencia con el modelo original y con el nivel de detalle requerido por el usuario final.

## Reporte transparente

El reporte debe describir claramente:

- población fuente;
- desenlace;
- predictores;
- manejo de faltantes;
- tamaño de muestra;
- especificación del modelo;
- validación;
- métricas de desempeño.

Las guías TRIPOD son una referencia central para este fin [@tripod2015].

## Recomendaciones

1. Reportar siempre discriminación y calibración.
2. No usar exactitud como métrica principal en datos desbalanceados.
3. Preferir *bootstrap* a una simple partición cuando el tamaño muestral es limitado.
4. Buscar validación externa antes de implementación amplia.
5. Evaluar utilidad clínica si el modelo pretende apoyar decisiones.
