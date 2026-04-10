
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
