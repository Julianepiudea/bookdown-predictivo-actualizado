# Bookdown: Modelado predictivo en salud

Este proyecto quedó organizado para compilarse como libro en HTML con **bookdown** y publicarse después en **GitHub Pages**.

## Estructura

- `index.Rmd`: portada, presentación y configuración inicial
- capítulos `01` a `11`: contenido principal del libro
- `_bookdown.yml`: orden de capítulos y carpeta de salida (`docs/`)
- `_output.yml`: formatos de salida
- `references.bib`: bibliografía básica
- `style.css`: estilos sencillos
- `.github/workflows/render-bookdown.yml`: flujo opcional para renderizar y publicar con GitHub Actions

## Cómo compilar localmente

```r
install.packages(c(
  "bookdown", "rmarkdown", "knitr", "tidyverse", "survival",
  "rms", "mice", "glmnet", "nnet", "ordinal", "riskRegression",
  "pec", "timeROC", "randomForestSRC", "cmprsk", "JMbayes2"
))

bookdown::render_book("index.Rmd", output_format = "bookdown::gitbook")
```

El resultado quedará en la carpeta `docs/`.

## Cómo subirlo a GitHub y obtener un link

### Opción manual
1. Crear un repositorio nuevo en GitHub.
2. Subir todos los archivos del proyecto.
3. Renderizar el libro localmente.
4. Confirmar que la carpeta `docs/` quedó actualizada.
5. En GitHub: **Settings > Pages**.
6. En **Build and deployment**, elegir:
   - **Source**: `Deploy from a branch`
   - **Branch**: `main`
   - **Folder**: `/docs`
7. Guardar. GitHub generará un enlace público.

### Opción automática con GitHub Actions
Este proyecto ya incluye un flujo de trabajo en `.github/workflows/render-bookdown.yml`.  
Si activas GitHub Pages en el repositorio, cada `push` a `main` podrá reconstruir el libro y publicarlo automáticamente.

## Sugerencias siguientes

- Agregar ejemplos reproducibles con bases de datos reales o simuladas.
- Incorporar figuras de calibración, ROC, Kaplan-Meier y DCA.
- Ajustar autoría, portada, logo institucional y colores.
- Ampliar citas bibliográficas según el curso o seminario.
