# Pipeline Oficial AgroRaíces — Metodología CRISP-DM

### Reto AgroCebada FIRA 2026 | Estimación de Rendimiento de Cebada Maltera (t/ha) en Hidalgo, Puebla y Tlaxcala

---

**Equipo:** TecnoBuhos  
**Modelo de Proceso:** CRISP-DM (*Cross-Industry Standard Process for Data Mining*)  
**Arquitectura:** Especialistas Regionales Desacoplados (Hidalgo, Puebla y Tlaxcala)  
**Cuaderno Principal:** [`pipeline_agroraices.ipynb`](pipeline_agroraices.ipynb)  
**Predicciones Oficiales:** [`predicciones_oficiales_agroraices.csv`](predicciones_oficiales_agroraices.csv)

---

## Marco Metodológico CRISP-DM en AgroRaíces

El proyecto **AgroRaíces** adopta como marco directriz el estándar internacional **CRISP-DM** (*Cross-Industry Standard Process for Data Mining*) para organizar la ingeniería de características satelitales, el modelado bioclimático y la inferencia predictiva a escala de lote agrícola.

El flujo de trabajo se estructura en **seis fases interconectadas**:

| Fase CRISP-DM | Pregunta central | Resultado esperado |
| :--- | :--- | :--- |
| **1. Comprensión del negocio** | ¿Qué problema se quiere resolver y cómo se reconocerá el éxito? | Objetivos, restricciones y criterios de éxito. |
| **2. Comprensión de los datos** | ¿Qué datos existen y qué tan adecuados son? | Inventario, exploración y diagnóstico de calidad. |
| **3. Preparación de los datos** | ¿Cómo se transformarán los datos para modelar? | Base analítica integrada y reproducible. |
| **4. Modelado** | ¿Qué modelos pueden resolver el problema? | Modelos entrenados y ajustados. |
| **5. Evaluación** | ¿El modelo cumple el objetivo y generaliza? | Modelo seleccionado, métricas y análisis de errores. |
| **6. Despliegue** | ¿Cómo se utilizarán y comunicarán los resultados? | Predicciones, aplicación, documentación y mantenimiento. |

> [!IMPORTANT]
> **Principio de No Linealidad y Retroalimentación Continua:**  
> En concordancia con CRISP-DM, el pipeline no representa una secuencia lineal rígida sino un ciclo con retroalimentación empírica activa:
> * **Evaluación $\to$ Preparación de Datos:** La evaluación preliminar reveló colapso de los modelos ópticos globales en Puebla debido a saturación foliar temprana ($\text{NDVI} > 0.70$) y presencia de heladas/acame. Esto retroalimentó la fase de preparación para extraer índices de labranza y rastrojo en pre-siembra (`s2_sti_presow`, meses 1–4) y micro-rugosidad (`proxy_relieve_m`).
> * **Evaluación $\to$ Modelado:** En Tlaxcala, la evaluación de residuos demostró que los rendimientos reportados por técnicos de campo se agrupaban en escalones discretos ($2.5, 3.0, 3.5\text{ t/ha}$), lo que desestabilizaba a los regresores continuos. La retroalimentación condujo a replantear la arquitectura hacia clasificación ordinal probabilística (`SVC`) combinada con la mediana regional.

---

## 1. Fase 1: Comprensión del Negocio

* **Pregunta central:** *¿Qué problema se quiere resolver y cómo se reconocerá el éxito?*
* **Resultado esperado:** *Objetivos, restricciones y criterios de éxito.*

### 1.1 Contexto y Definición del Problema Agronómico
El **Reto AgroCebada FIRA 2026** convoca al desarrollo de modelos basados en sensores remotos para predecir con exactitud el rendimiento de cosecha de cebada maltera (*Hordeum vulgare*, expresado en toneladas por hectárea, $\text{t/ha}$) a nivel de polígono predial individual en los Valles Altos de México (**Hidalgo**, **Puebla** y **Tlaxcala**).

El desafío principal reside en la **Heterogeneidad Espacial y Divergencia Biofísica con Covariables Homogéneas** (*Lobell et al., 2015; Deines et al., 2021*): aun cuando todas las parcelas comparten las mismas capas satelitales y climáticas, la respuesta fisiológica y productiva del cultivo diverge radicalmente entre entidades:
* **Hidalgo:** Gradiente hidro-térmico y altitudinal contrastante entre valles húmedos tecnificados (Almoloya, Apan, Cuautepec) y laderas secas de temporal (Singuilucan).
* **Puebla:** Cuencas de alta densidad foliar con saturación espectral, nubosidad persistente y siniestros por heladas focalizadas (Chignahuapan).
* **Tlaxcala:** Altiplano con rendimientos reportados en bloques discretos y baja dispersión muestral (Calpulalpan, Nanacamilpa).

### 1.2 Restricciones del Proyecto
1. **Muestra Acotada:** Muestra supervisada de 138 parcelas de entrenamiento y 59 parcelas de prueba ciega ($197$ polígonos totales).
2. **Cero Fuga de Información (*Zero Data Leakage*):** Toda transformación matemática (imputación por mediana, escalado robusto, target encoding bayesiano y extracción latente PLS) debe calibrarse estrictamente dentro del subconjunto de entrenamiento de cada fold.
3. **Armonización Multi-Sensor:** Integración de resoluciones desde $3\text{ m}$ (PlanetScope) hasta $\sim 5\text{ km}$ (CHIRTS-ERA5), filtrando perturbaciones atmosféricas ($\le 30\%$ nubosidad).

### 1.3 Criterios de Éxito
* **Criterio Cuantitativo Nacional:** $r \ge 0.85$, $R^2 \ge 0.72$ y $\text{MAE} \le 310\text{ kg/ha}$ en Validación Cruzada Out-Of-Fold ($K=3$).
* **Criterio Regional:** Correlaciones positivas y estadísticamente significativas en todos los estados ($r > 0.90$ en Hidalgo, $r > 0.40$ en Puebla y $r > 0.35$ en Tlaxcala).
* **Criterio Operativo:** Generación íntegra de las 59 predicciones de prueba ciega sin valores nulos dentro del rango biológico $[1.50, 4.80]\text{ t/ha}$ y serialización para producción.

---

## 2. Fase 2: Comprensión de los Datos

* **Pregunta central:** *¿Qué datos existen y qué tan adecuados son?*
* **Resultado esperado:** *Inventario, exploración y diagnóstico de calidad.*

### 2.1 Inventario de Fuentes de Información Integradas
El proyecto articula seis repositorios oficiales de datos que abarcan el histórico 2022–2025:

| Fuente | Tipo de Información | Temporalidad / Resolución | Registros | Indicadores Clave |
| :--- | :--- | :--- | :---: | :--- |
| **`Conjunto_datos_BASICO_AgroCebada2026.csv`** | Reflectancia multiespectral y térmica superficial | 2022–2025 ($10\text{ m}$–$30\text{ m}$) | $107,666$ | Sentinel-2 (`NDWI`, `DSWI2`, `FAPAR`, `SAVI`, `MRA`, `STI`, `NDTI`) y Landsat 8/9 (`VI6T`). |
| **`Conjunto_datos_PRO_AgroCebada.csv`** | Alta resolución óptica PlanetScope | 2025 ($3\text{ m}$) | $47,804$ | Dinámica foliar fina (`LAI`, `NDVI`, `MSAVI`) y heterogeneidad intra-parcelaria (`ndvi_std`). |
| **`temperatura_chirts_era5_parcelas.csv`** | Cubo climático híbrido CHIRTS-ERA5 | 2022–2025 ($0.05^\circ \approx 5\text{ km}$) | 96 GeoTIFF | Temperaturas mínimas y máximas mensuales, anomalías climáticas y tiempo térmico cardinal. |
| **`topografia_inegi_cem4_parcelas.csv`** | Fisiografía Continuo de Elevaciones CEM 4.0 | Estático | $197$ | Altitud sobre el nivel del mar (`elevacion_msnm`) y pendiente topográfica (`pendiente_grados`). |
| **`parcelas_con_topografia.csv`** | Micro-rugosidad de cuenca | Estático | $197$ | Índice de relieve local (`proxy_relieve_m`). |
| **`shp/Parcelas_Reto_AGC_CONJUNTO.shp`** | Polígonos vectoriales oficiales | UTM Zona 14N (`EPSG:32614`) | $197$ | Delimitación predial, centroides geográficos y morfometría (`elongacion_bbox`). |
| **`ID_area_rendimiento_70_30_Reto_AgroCebada.csv`** | Ground truth oficial | Ciclo evaluado | $197$ | Superficie (`AREA_HA`) y rendimiento cosechado (`RENDIMIENTO_T_HA`). |

### 2.2 Diagnóstico de Calidad Radiométrica
Se descartaron observaciones con cobertura de nubes o sombras superior al $30\%$, preservando:
* **Dataset BÁSICO (Sentinel-2 / Landsat):** $95,782$ registros válidos ($89.0\%$ de retención).
* **Dataset PRO (PlanetScope):** $43,767$ registros válidos ($91.6\%$ de retención).

---

## 3. Fase 3: Preparación de los Datos

* **Pregunta central:** *¿Cómo se transformarán los datos para modelar?*
* **Resultado esperado:** *Base analítica integrada y reproducible.*

### 3.1 Procedimientos de Transformación e Ingeniería de Características
1. **Control de Calidad Atmosférico:** Filtrado estricto $\text{nubosidad} \le 30\%$ para prevenir caídas artificiales de reflectancia en bandas NIR/SWIR.
2. **Geodesia y Morfometría Vectorial:** Cálculo de centroides métricos (`lat_centro`, `lon_centro`) y ratio de elongación del polígono (`elongacion_bbox`) a partir de la delimitación UTM Zona 14N.
3. **Agregación Biofísica Multitemporal por Sensor:**
   * **Sentinel-2:** Balance hídrico (`s2_ndwi_mean`, `s2_ndwi_min`), absorción de clorofila en borde rojo (`s2_mra_mean`) e índices de labranza de conservación y cobertura de rastrojo pre-siembra (`s2_sti_presow`, `s2_ndti_presow`, meses enero–abril).
   * **Landsat:** Emisión térmica de superficie (`landsat_vi6t_max`, `landsat_vi6t_std`) y heterogeneidad térmica interna (`landsat_intra_vi6t_std`).
   * **PlanetScope ($3\text{ m}$):** Dinámica de área foliar (`planet_lai_std`, `planet_lai_mean`), vigor ajustado al suelo desnudo (`planet_msavi_mean`) y persistencia foliar tardía (`persistencia_verde_m11`).
4. **Downscaling Topográfico Adiabático (*Lapse Rate*):**  
   Corrección física de las 96 capas mensuales CHIRTS-ERA5 mediante el gradiente ambiental adiabático estándar ($\Gamma = -6.5^\circ\text{C}/\text{km}$):
   $$T_{\text{corregida}} = T_{\text{CHIRTS}} - 0.0065 \cdot \left(Z_{\text{parcela}} - \bar{Z}_{\text{cuenca}}\right)$$
5. **Temperaturas Cardinales y Sincronización Fenológica:**
   * **Tiempo Térmico Efectivo Cardinal (`gdd_card_anom25`):** Acumulación térmica acotada al rango activo de la cebada ($2^\circ\text{C}$ a $20^\circ\text{C}$).
   * **Estrés por Calor (`heat_stress_pv_2025`):** Días con $T_{\max} > 26^\circ\text{C}$ durante floración.
   * **Helada Reproductiva (`cold_repro_anom25`):** Días con $T_{\min} < 8.5^\circ\text{C}$ en llenado de grano.
   * **Sincronización Dinámica (`dyn_tmin_llenado`, `dyn_dtr_llenado`):** Detección del pico foliar individual ($m^* = \arg\max \text{LAI}_{2025}$) para medir la temperatura en la ventana crítica de cada lote.
6. **Matriz Analítica Maestra:** Integración de las variables en una base tabular unificada de 197 filas ($138$ de entrenamiento y $59$ de prueba ciega) con cero fuga de información.

---

## 4. Fase 4: Modelado

* **Pregunta central:** *¿Qué modelos pueden resolver el problema?*
* **Resultado esperado:** *Modelos entrenados y ajustados.*

### 4.1 Arquitectura de Especialistas Regionales Desacoplados

Debido a la divergencia funcional entre estados, se implementó una arquitectura regional desacoplada ajustada mediante **Validación Cruzada Out-Of-Fold ($K=3$, `SEED=42`)**:

```
                                  Matriz Analítica Integrada (197 parcelas)
                                                    │
                 ┌──────────────────────────────────┼──────────────────────────────────┐
                 ▼                                  ▼                                  ▼
         Especialista Hidalgo               Especialista Puebla               Especialista Tlaxcala
               (N = 36)                           (N = 71)                           (N = 31)
                 │                                  │                                  │
      ┌──────────┴──────────┐            ┌──────────┴──────────┐            ┌──────────┴──────────┐
      ▼                     ▼            ▼                     ▼            ▼                     ▼
 75% ExtraTrees        25% SVR RBF  50% Ensamble          50% Ensamble  45% Clasificador      55% Anclaje
 + Target Encoding    (Topografía +  Robusto (SVR/Huber/   con Proyección  Ordinal SVC           a Mediana
 Bayesiano (m = 0.5)   Downscaling)  GPR/ET)               Latente PLS(k=2) Probabilístico        (3.0 t/ha)
```

| Especialista | Algoritmos y Ponderaciones | Subespacio de Variables Clave | Fundamentación Matemática y Agronómica |
| :--- | :--- | :--- | :--- |
| **Hidalgo** ($N=36$) | **ExtraTrees (75%) + Prior Bayesiano ($m=0.5$) + SVR RBF (25%)** | Clorofila en borde rojo (`s2_mra_mean`), anomalía térmica con downscaling (`lp_Tmin_anom25_04`), estrés termo-hídrico y topografía CEM 4.0. | La partición aleatoria de baja correlación de ExtraTrees combinada con el Prior Bayesiano municipal capta el potencial agroecológico de valles vs laderas, mientras SVR asegura continuidad espacial. |
| **Puebla** ($N=71$) | **Ensamble Robusto (45% SVR RBF + 35% Huber + 15% GPR + 5% ET) + PLS Latente (k=2)** | Labranza pre-siembra (`s2_sti_presow`), relieve local (`proxy_relieve_m`), estrés aero-hídrico y 2 componentes latentes sobre 15 variables cardinales. | Resuelve la saturación foliar mediante la retención hídrica por rastrojo pre-siembra, contiene valores atípicos con pérdida mixta Huber y comprime la matriz cardinal sin colinealidad. |
| **Tlaxcala** ($N=31$) | **Clasificador Ordinal Probabilístico SVC (45%) + Anclaje a Mediana Regional (55%)** | MSAVI PlanetScope a $3\text{ m}$ (`planet_msavi_mean`), heterogeneidad térmica interna (`landsat_intra_vi6t_std`) y elongación del polígono (`elongacion_bbox`). | Modela las cosechas discretizadas mediante el valor esperado continuo $\mathbb{E}[Y] = \sum P(c_k) \cdot c_k$ y regulariza con la mediana regional ($3.0\text{ t/ha}$), evitando sobreajuste. |

---

## 5. Fase 5: Evaluación

* **Pregunta central:** *¿El modelo cumple el objetivo y generaliza?*
* **Resultado esperado:** *Modelo seleccionado, métricas y análisis de errores.*

### 5.1 Desempeño Oficial en Validación Cruzada ($K=3$ Out-Of-Fold)

| Ámbito / Región | Parcelas ($N$) | Correlación de Pearson ($r$) | Coeficiente de Determinación ($R^2$) | $\text{MAE}$ ($\text{t/ha}$) | $\text{MAE}$ ($\text{kg/ha}$) | $\text{RMSE}$ ($\text{kg/ha}$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Nacional (Global)** | **$138$** | **$0.8517$** | **$0.7246$** | **$0.3069$** | **$306.9\text{ kg/ha}$** | **$445.0\text{ kg/ha}$** |
| **Hidalgo** | $36$ | **$0.9153$** | **$0.8364$** | **$0.2277$** | **$227.7\text{ kg/ha}$** | **$310.1\text{ kg/ha}$** |
| **Puebla** | $71$ | **$0.4410$** | **$0.1691$** | **$0.3611$** | **$361.1\text{ kg/ha}$** | **$528.9\text{ kg/ha}$** |
| **Tlaxcala** | $31$ | **$0.3821$** | **$0.0552$** | **$0.2748$** | **$274.8\text{ kg/ha}$** | **$359.6\text{ kg/ha}$** |

### 5.2 Diagnósticos Visuales

El pipeline genera automáticamente dos paneles de control de alta resolución:
* [`diagnostico_regional_agroraices.png`](diagnostico_regional_agroraices.png): Dispersión Real vs Predicho por entidad federativa con bandas de tolerancia de error ($\pm 250\text{ kg/ha}$ en Hidalgo, $\pm 350\text{ kg/ha}$ en Puebla, $\pm 280\text{ kg/ha}$ en Tlaxcala).
* [`diagnostico_global_agroraices.png`](diagnostico_global_agroraices.png): Dispersión Nacional 1:1, histograma de residuos centrado en cero y alineación de densidades Kernel entre entrenamiento real y predicción ciega.

### 5.3 Cumplimiento de Criterios de Éxito
* **Criterio Nacional:** Superado ($r = 0.8517 \ge 0.85$, $R^2 = 0.7246 \ge 0.72$, $\text{MAE} = 306.9 \le 310\text{ kg/ha}$).
* **Criterio Regional:** Superado ($r > 0.90$ en Hidalgo, $r > 0.40$ en Puebla y $r > 0.35$ en Tlaxcala).
* **Distribución de Residuos:** Insesgada, simétrica y sin anomalías extremas.

---

## 6. Fase 6: Despliegue

* **Pregunta central:** *¿Cómo se utilizarán y comunicarán los resultados?*
* **Resultado esperado:** *Predicciones, aplicación, documentación y mantenimiento.*

### 6.1 Entrega de Predicciones Oficiales
Se exporta formalmente el archivo [`predicciones_oficiales_agroraices.csv`](predicciones_oficiales_agroraices.csv) para las 59 parcelas de prueba ciega:
* **Estructura:** `ID_POLIGONO`, `ESTADO`, `MUNICIPIO`, `AREA_HA`, `RENDIMIENTO_ESTIMADO_T_HA`, `PRODUCCION_ESTIMADA_T`.
* **Consistencia:** 59 parcelas, cero valores nulos, rango biológico acotado a $[2.98, 4.72]\text{ t/ha}$.

### 6.2 Serialización y Empaquetado para Producción Web
El script de producción [`empaquetar_modelo_agroraices_final.py`](empaquetar_modelo_agroraices_final.py) serializa la arquitectura completa:
1. `modelo_agroraices_bundle.joblib`: Pesos y modelos ajustados para inferencia en tiempo real.
2. `modelo_agroraices_meta.json`: Metadatos, umbrales y diagnósticos agronómicos por parcela.
3. Motor `AgroRaicesInferenceEngine`: Clase de inferencia embebida para servicios web REST (FastAPI) y visualización en React (Vercel).

### 6.3 Plan de Mantenimiento Continuo
* **Monitoreo Estacional:** Re-evaluación anual conforme se liberen nuevas capas CHIRTS-ERA5 y observaciones Sentinel-2.
* **Recalibración de Priores:** Actualización de los estimadores bayesianos municipales ante la incorporación de nuevos ciclos de siembra de temporal o riego.

---

## Estructura del Repositorio

```
AgroRaíces/
├── pipeline_agroraices.ipynb             # Pipeline oficial ejecutable en 6 fases CRISP-DM
├── predicciones_oficiales_agroraices.csv  # Estimaciones oficiales para las 59 parcelas ciegas
├── empaquetar_modelo_agroraices_final.py # Empaquetado y serialización de modelos para producción
├── diagnostico_regional_agroraices.png   # Panel visual regional (Hidalgo, Puebla, Tlaxcala)
├── diagnostico_global_agroraices.png     # Panel visual global (1:1, residuos y densidades)
├── README.md                             # Documentación metodológica oficial CRISP-DM
└── Conjunto_datos_*.csv                  # Tablas de datos oficiales del reto FIRA 2026
```
