
---
# 3. Herramientas implementadas

## 3.1. Elementos de programación

### 3.1.1. Web Scraping

La recolección automatizada de datos se realizó sobre dos portales:

**Portal de microdatos del INEI** - Módulos ENAHO (2020–2024)

| Módulo | Contenido | Formato |
|--------|-----------|---------|
| Módulo 200 | Vivienda y hogar | `.zip` / `.csv` |
| Módulo 300 | Educación | `.zip` / `.csv` |
| Módulo 34 | Sumarias (variables resumen) | `.zip` / `.csv` |
| Módulo 400 | Salud | `.zip` / `.csv` |

- Herramienta: `requests` + `zipfile` + `io`
- Se identificó un patrón de URLs estructuradas inspeccionando el código fuente con el inspector F12 del navegador (DevTools de Chrome)
- URL base: `https://proyectos.inei.gob.pe/iinei/srienaho/descarga/CSV/{ID}-Modulo{MOD}.zip`
- Se iteró sobre 5 años × 4 módulos (200, 300, 34, 400) mediante un bucle anidado con manejo de errores
- Los archivos `.zip` se descomprimieron en memoria con `zipfile.ZipFile(io.BytesIO(...))` extrayendo únicamente los `.csv`

> **Nota sobre el script de Web Scraping:** El archivo 
> `scripts/WEB_SCRAPING.ipynb` forma parte de la Tarea Calificada 2
> del curso, en la que se automatizó la descarga de los módulos ENAHO 
> (portal del INEI) y el archivo IDH (portal del PNUD Perú) mediante 
> técnicas de web scraping. Este script se incluye en el repositorio 
> como evidencia de dicho proceso. El flujo principal del proyecto 
> corre desde `0_MASTER_SCRIPT.ipynb`.

**PNUD Perú** - Anexo 1: IDH 2017-2024 a nivel distrital
- Herramienta: `requests` con headers de navegador
- URL directa identificada inspeccionando el atributo `href` del botón de descarga (clase `download-btn`) con F12
- Se agregaron headers `User-Agent`, `Referer` y `Accept` para evitar el bloqueo HTTP 403 del servidor
- Archivo descargado: `anexo_1-idh_2017-2024_a_nivel_distrital.xlsx`

## 3.1.2. Manejo de bases de datos
El procesamiento de datos se realizó íntegramente en Python con `pandas`:

**Carga y limpieza**
- Lectura de archivos `.csv` con encoding `latin-1` (requerido por los archivos ENAHO)
- Conversión de variables a tipo numérico con `pd.to_numeric(..., errors='coerce')`
- Imputación de valores faltantes con `.fillna(0)`
- Normalización de nombres de columnas con `.str.lower()`

**Construcción de variables**
- `gasto_bolsillo`: suma ponderada de montos anuales por rubro (consultas, medicinas, análisis, etc.) condicionada al indicador de pago del hogar (`p4151_XX == 1`)
- `tiene_seguro`: variable binaria construida con `.any(axis=1)` sobre las columnas de afiliación (`p4191`–`p4198`)
- `enfermo_4sem`: variable binaria de morbilidad en las últimas 4 semanas

**Integración de fuentes**
- Merge de los 4 módulos ENAHO usando llaves `conglome`, `vivienda`, `hogar`, `codperso`
- Merge con el IDH provincial usando `ubigeo` como llave de unión
- Consolidación de 5 años (2020–2024) con `pd.concat(..., ignore_index=True)`

## 3.1.3. Gráficos y visualización
Las visualizaciones se produjeron con `matplotlib` y `geopandas`:

- Gráficos de evolución temporal del gasto de bolsillo por año
- Distribuciones del GBS por tipo de seguro y quintil de ingreso
- Mapas distritales del GBS y el IDH usando `geopandas` con shapefiles del INEI
- Tablas de valores anuales con población expandida usando el factor de expansión `factor07`

## 3.1.4. Análisis estadístico
El análisis econométrico se realizó con `statsmodels`:

- Modelos de regresión OLS para estimar la asociación entre nivel educativo, IDH provincial y gasto de bolsillo en salud
- Variable dependiente: `gasto_bolsillo` (monto anual en soles)
- Variables independientes principales: nivel educativo individual e IDH provincial
- Control por tipo de seguro, condición de morbilidad y año
  
## 3.2. Herramientas de IA Generativa

En el desarrollo de este proyecto, las herramientas de IA generativa pueden cumplir un rol importante en distintas etapas del flujo de trabajo, desde la planificación del código hasta su revisión, documentación y mejora. Asistentes como Gemini in Colab, Gemini Code Assist, Claude Code y GitHub Copilot permiten generar fragmentos de código, sugerir estructuras para funciones, explicar errores y proponer mejoras en la lógica del programa. Estas herramientas no reemplazan el razonamiento del equipo, pero sí agilizan tareas operativas y facilitan que el equipo concentre más esfuerzo en la calidad metodológica del análisis y en la interpretación de resultados. 

A continuación se detalla cómo podrían aplicarse en cada sección del proyecto:

**Carga y Limpieza de datos:** Gemini in Colab podría asistir en la construcción de variables derivadas como `gasto_bolsillo` o `tiene_seguro`, sugiriendo la lógica de agregación más adecuada. Claude Code podría apoyar en la depuración del merge entre módulos y la identificación de inconsistencias en las llaves de unión (`conglome`, `vivienda`, `hogar`, `codperso`). 

**Estadística Descriptiva:** Claude o Gemini podrían apoyar en la redacción de las interpretaciones de resultados, como la distribución asimétrica del gasto de bolsillo o el patrón por nivel educativo, mejorando su claridad y precisión académica. GitHub Copilot o Gemini Code Assist podrían sugerir funciones para calcular estadísticos ponderados usando el factor de expansión `factor07`. 

**Visualización de datos:** Gemini in Colab podría sugerir paletas de colores, ajustes de escala y etiquetas que mejoren la legibilidad de los gráficos de barras y líneas de tendencia. Para el mapa coroplético de evolución del IDH provincial, Claude podría orientar sobre el uso de `geopandas` y la selección del esquema de clasificación más adecuado. 

**Análisis predictivo:** Claude Code podría apoyar en la especificación del modelo log-lineal y en la interpretación de coeficientes como el del IDH (β = 2.78), cuyo signo positivo resulta contraintuitivo y requiere una lectura cuidadosa. Gemini podría asistir en la redacción de los hallazgos para el informe final, traduciendo los resultados estadísticos a un lenguaje accesible para audiencias de política pública. 

**Documentación y presentación:** Para la redacción del README, los comentarios del código y la presentación final, pueden emplearse herramientas como Claude o Gemini para mejorar la claridad, organización y estilo académico del texto. Como herramienta complementaria, Black puede emplearse para estandarizar el formato del código en todos los notebooks, mejorando su legibilidad y consistencia, aunque no constituye una herramienta de IA generativa.
