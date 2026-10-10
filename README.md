# 🛰️ Pipeline Predictivo de Inteligencia Satelital SAOCOM - Establecimiento "La Rosaura"

Este repositorio aloja el desarrollo del proyecto final para el despliegue de un pipeline unificado de Machine Learning e Inteligencia Geoespacial. El sistema está diseñado para el análisis histórico y el monitoreo en tiempo real del cultivo de maíz de primera destinado a la confección de **silo forrajero de planta entera** en el establecimiento lechero (tambo) **"La Rosaura"**, ubicado en el partido de Tandil, Provincia de Buenos Aires.

El pipeline procesa la información utilizando datos reales de Radar de Apertura Sintética (SAR) en **Banda L** provenientes de la constelación argentina **SAOCOM 1A y 1B (CONAE)**. Al operar en microondas, el sistema evita la restricción por cubierta nubosa estival en Tandil y penetra la canopia, midiendo la constante dieléctrica del suelo y la biomasa estructural por **Unidad Espacial a su resolución nativa de 10 metros por píxel**.

---

## 📌 1. Formulación del Problema y Tareas de Machine Learning (Unidad 2)

El sistema aborda el soporte de decisiones agronómicas, operativas y financieras mediante un esquema supervisado configurado bajo tres enfoques analíticos complementarios:

1. **Enfoque de Regresión (Misión Principal):** Estimación continua de la biomasa proyectada en Toneladas por Hectárea (**TN/HA**) de materia verde a partir del perfil hídrico de siembra y la evolución del RVI (Rango histórico: **38.16 a 49.71 TN/HA**).
2. **Enfoque de Clasificación Binaria I (Ventana Óptima de Siembra - ¡NUEVO!):** Clasificación del estado térmico e hídrico de la cama de siembra (**1** si el píxel presenta condiciones hidrodinámicas seguras en el canal HH, **0** si se encuentra bloqueado por riesgo de secado de la semilla o anegamiento).
3. **Enfoque de Clasificación Binaria II (Ventana Óptima de Picado):** Alerta temprana de cosecha (**1** si el lote ingresa en la ventana crítica del **32% al 35% de Materia Seca (MS)** para garantizar la correcta fermentación y calidad nutricional del silo, **0** si está fuera de rango).

### 🎯 Variables Objetivo (Targets)
* `rendimiento_tn_ha`: Variable continua de biomasa estimada (Regresión).
* `ventana_optima_siembra`: Variable categórica binaria (0/1) para habilitación de siembra (Clasificación).
* `ventana_optima_ms`: Variable categórica binaria (0/1) para alerta de picado oportuno (Clasificación).

---

## 🔬 2. Especificaciones de la Ingesta de Datos Reales (CONAE)

El pipeline procesa un dataset histórico compuesto por **167 escenas satelitales reales** (período 2020 - Actualidad), estructurado bajo los siguientes pilares de dominio espacial:
* **Nivel de Procesamiento L1D:** Las imágenes corresponden al producto ortorrectificado de la CONAE, completamente planchadas sobre la topografía real del terreno, lo que permite que la máscara geográfica calce de forma milimétrica sobre las matrices numéricas.
* **Polarización Cuádruple QP (Quad-Pol):** Extracción de los canales **HH** (retrodispersión sensible a la constante dieléctrica del agua en el suelo) y **HV** (sensible a la dispersión volumétrica y densidad de tallos erectos del maíz).
* **Ingeniería de Atributos Avanzada:** El script deriva linealmente el **Índice de Vegetación de Radar (RVI = 8*HV / (HH+HV))**, ratios de despolarización cross-pol (`HV/HH`) y tasas temporales de deshidratación a diferencias finitas de 7 días.

---

## 🚜 3. Módulo de Fertilización Selectiva Variable a Resolución Nativa (10m)

Para mitigar el lavado de nitrógeno por escorrentía en lomas altas y aprovechar el máximo potencial de los bajos hidrológicos del establecimiento, el pipeline modela las prescripciones a nivel de píxel individual (mínimo nivel de desagregación física permitido por el satélite). 

El script procesa las desviaciones de vigor relativo (`desvio_rvi_vs_historico`) en estadios fenológicos tempranos (**V4 - V8**) y exporta de forma automatizada un archivo vectorial georreferenciado en formato **Shapefile comprimido (`prescripcion_variable.zip`)**. Este archivo contiene los atributos métricos indexados (`pixel_id`, `lat_dec`, `lon_dec`, `dosis_nitrogeno`) para ser interpretados de forma nativa por las pantallas de las maquinarias fertilizadoras (por ejemplo: John Deere, Trimble, Abelardo Cuffia), regulando la dosificación electrónica de urea cada 10 metros en tiempo real.

---

## 📊 4. Arquitectura del Repositorio y Secuencia de Unidades

El proyecto sigue una estructura modular alineada con los requisitos exigidos en la materia:
* `README.md`: Portada metodológica y flujo del negocio.
* `changelog.md`: Registro cronológico e histórico de evolución del pipeline.
* `La_rosaura.py`: Script de consola para el procesamiento hídrico y topográfico.
* `catalogo_saocom.csv`: Base de datos real indexada con las 167 pasadas de la CONAE.
* `data/`: Carpeta local que aloja las matrices de salida y el mapa hídrico `mapa_humedad_pixel.png`.

---

## 🚀 5. Instrucciones de Ejecución Local

1. Clonar el repositorio y activar el entorno virtual aislado:
   ```bash
   git clone <URL_DE_TU_REPOSITORIO>
   cd la_rosaura
   python -m venv .venv
   .venv\Scripts\activate
   ```
2. Instalar el stack de librerías de Ciencia de Datos:
   ```bash
   pip install numpy shapely matplotlib rasterio pandas scikit-learn
   ```
3. Ejecutar el pipeline unificado de regresión y clasificación:
   ```bash
   python La_rosaura.py
   ```
