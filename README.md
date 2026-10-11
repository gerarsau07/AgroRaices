# Pipeline Oficial AgroRaíces — Predicción Geoespacial, Climática y Biofísica de Rendimiento de Cebada Maltera

**Reto AgroCebada 2026 | Estimación de Rendimiento Agrícola (t/ha) en Hidalgo, Puebla y Tlaxcala**

---

## Resumen Ejecutivo

Este repositorio contiene el pipeline científico y de producción desarrollado para estimar el rendimiento de **cebada maltera (t/ha)** a nivel de parcela en los Valles Altos de México (**Hidalgo, Puebla y Tlaxcala**).

El modelo resuelve el problema conocido en la literatura de teledetección agrícola como **Predicción de Rendimiento bajo Heterogeneidad Espacial y Divergencia Biofísica con Covariables Homogéneas** (*Lobell et al., 2015; Deines et al., 2021; Wolanin et al., 2020*): aunque todas las parcelas comparten las mismas variables satelitales, topográficas y climáticas, la respuesta fisiológica del cultivo cambia radicalmente entre estados.

Mediante la integración de **6 fuentes de datos multitemporales (2022–2025)**, el pre-procesamiento físico de **96 capas climáticas GeoTIFF (CHIRTS-ERA5)** con *downscaling* adiabático y una arquitectura de **Especialistas Regionales Desacoplados**, el modelo alcanza en validación cruzada estricta ($K=3$ Out-Of-Fold):

* **Correlación Lineal de Pearson Nacional ($r$):** **`0.8517` – `0.8535`**
* **Coeficiente de Determinación Nacional ($R^2$):** **`0.7246` – `0.7276`** *
* **Error Absoluto Medio Nacional ($\text{MAE}$):** **`306.9 kg/ha`** (`0.3069 t/ha`)
* **Raíz del Error Cuadrático Medio ($\text{RMSE}$):** **`442.7` – `445.0 kg/ha`**

---

##  Resultados Oficiales en Validación Cruzada ($K=3$ Out-Of-Fold)

| Ámbito / Región | Parcelas ($N$) | Correlación de Pearson ($r$) | Coeficiente de Determinación ($R^2$) | $\text{MAE}$ ($\text{kg/ha}$) | $\text{RMSE}$ ($\text{kg/ha}$) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Nacional (Global)** | **$138$** | **$0.8517$** | **$0.7246$** | **$306.9\text{ kg/ha}$** | **$445.0\text{ kg/ha}$** |
| **Hidalgo** | $36$ | **$0.9153$** | **$0.8364$** | **$227.7\text{ kg/ha}$** | **$310.1\text{ kg/ha}$** |
| **Puebla** | $71$ | **$0.4410$** | **$0.1691$** | **$361.1\text{ kg/ha}$** | **$528.9\text{ kg/ha}$** |
| **Tlaxcala** | $31$ | **$0.3821$** | **$0.0552$** | **$274.8\text{ kg/ha}$** | **$359.6\text{ kg/ha}$** |

---

## Fuentes de Datos Integradas (2022–2025)

El pipeline fusiona información óptica, infrarroja de onda corta (SWIR), térmica superficial, climática, fisiográfica y geométrica de **197 polígonos agrícolas** ($138$ de entrenamiento y $59$ de prueba ciega):

1. **`Conjunto_datos_BASICO_AgroCebada2026.csv` ($107,666$ registros, 2022–2025):**
   * **Sentinel-2 ($10\text{ m}$–$20\text{ m}$):** Índices hídricos (`NDWI`, `DSWI2`), radiación fotosintéticamente activa no saturable (`FAPAR`, `SAVI`), absorción de clorofila en borde rojo (`MRA`), heterogeneidad interna (`evi_std`) e índices de **labranza de conservación y cobertura de rastrojo en pre-siembra** (`STI`, `NDTI`).
   * **Landsat 8/9 ($30\text{ m}$ térmico):** Temperatura de brillo superficial (`VI6T` máximo, promedio, desviación multitemporal y desviación espacial intra-parcelaria `vi6t_std`).
2. **`Conjunto_datos_PRO_AgroCebada.csv` ($47,804$ registros, 2025):**
   * **PlanetScope ($3\text{ m}$ de resolución espacial):** Dinámica foliar fina (`LAI`, `NDVI`), vigor ajustado al suelo desnudo (`MSAVI`), heterogeneidad interna (`ndvi_std`) y persistencia de verdor al cierre del ciclo (noviembre).
3. **`Reto_AgroCebada_Temperatura_2022_2025/` $\to$ `temperatura_chirts_era5_parcelas.csv`:**
   * **96 capas GeoTIFF mensuales ($T_{\min}$ y $T_{\max}$, 2022–2025)** del producto híbrido satelital-estación CHIRTS-ERA5 ($0.05^\circ \approx 5\text{ km}$, EPSG:4326), extraídas mediante interpolación bilineal sub-píxel y muestreo por vecino más cercano.
4. **`topografia_inegi_cem4_parcelas.csv` y `parcelas_con_topografia.csv`:**
   * Elevación sobre el nivel del mar (`elevacion_msnm`), pendiente topográfica (`pendiente_grados`) del Continuo de Elevaciones Mexicano (INEGI CEM 4.0) e índice de rugosidad/relieve local (`proxy_relieve_m`).
5. **`shp/Parcelas_Reto_AGC_CONJUNTO.shp`:**
   * Geometría vectorial de las 197 parcelas proyectada a coordenadas métricas UTM Zona 14N (`EPSG:32614`) para calcular centroides y la relación de elongación geométrica del lote (`elongacion_bbox`).
6. **`ID_area_rendimiento_70_30_Reto_AgroCebada.csv`:**
   * Superficie oficial (`AREA_HA`), partición `ENTRENAMIENTO` / `PREDICCION` y rendimiento cosechado (`RENDIMIENTO_T_HA`).

---

## Ingeniería de Características Físicas, Agroclimáticas y Edáficas

### 1. Control de Calidad Atmosférico ($\text{Nubosidad} \le 30\%$)
Se descartan todas las escenas con cobertura nubosa superior al $30\%$, eliminando sombras de nubes y reflectancias espurias que deprimen artificialmente el `NDVI` y el `NDWI`.

### 2. *Downscaling* Topográfico Adiabático de Temperatura (*Lapse Rate*)
Dado que la malla de CHIRTS-ERA5 tiene una resolución de $\sim 5\text{ km}$, parcelas vecinas con desniveles de hasta $300\text{ m}$ compartirían el mismo píxel climático. Se aplica la corrección física del gradiente adiabático ambiental estándar ($\Gamma = -6.5^\circ\text{C} / 1000\text{ m}$) usando el desnivel de cada parcela respecto a su cuenca ($\Delta Z$):

$$T_{\text{corregida}} = T_{\text{CHIRTS}} - 0.0065 \cdot \left(Z_{\text{parcela}} - \bar{Z}_{\text{cuenca}}\right)$$

### 3. Temperaturas Cardinales No Lineales de la Cebada
En lugar de asumir un crecimiento lineal indefinido con la temperatura, se modelan los umbrales fisiológicos reales de la cebada maltera:
* **Tiempo Térmico Efectivo Cardinal (`gdd_card_anom25`):** Acumulación térmica acotada entre $T_{\text{base}} = 2^\circ\text{C}$ y $T_{\text{óptima}} = 20^\circ\text{C}$.
* **Estrés por Golpe de Calor en Antesis (`heat_stress_pv_2025`):** Exceso térmico diurno cuando $T_{\max} > 26^\circ\text{C}$, responsable de senescencia prematura y reducción del peso hectolítrico.
* **Riesgo de Helada/Frío Reproductivo (`cold_repro_anom25`):** Exposición acumulada a temperaturas mínimas nocturnas $< 8.5^\circ\text{C}$ entre agosto y octubre (etapa de espigamiento y llenado de grano).

### 4. Sincronización Fenológica Dinámica y Balance Dosel-Aire
* **Fenología Dinámica (`dyn_tmin_llenado`, `dyn_dtr_llenado`):** Se detecta en PlanetScope ($3\text{ m}$) el mes exacto del pico foliar individual ($m^* = \arg\max \text{LAI}_{2025}$) de cada parcela y se extrae el régimen térmico del mes siguiente ($m^* + 1$, llenado de grano), desacoplando el clima del calendario fijo.
* **Diferencial Dosel-Aire Sincronizado (`sync_canopy_ratio_mean`, `bi_canopy_air_diff`):** Empareja cada pasada orbital de Landsat (`VI6T`) con la temperatura del aire CHIRTS-ERA5 del mismo mes y año, distinguiendo cultivos con transpiración activa (dosel frío) de aquellos con cierre estomático por estrés hídrico (dosel caliente).

### 5. Indicadores Edáficos de Labranza Pre-Siembra y Morfometría
* **Labranza de Conservación y Cobertura de Rastrojo (`s2_sti_presow`, `s2_ndti_presow`):** Calculados sobre las bandas SWIR de Sentinel-2 entre enero y abril (pre-siembra). En **Puebla**, donde el `NDVI` vegetativo se satura ($r \approx -0.03$), el índice de labranza pre-siembra alcanza la correlación positiva más alta del estado (**$r = +0.3228$**), capturando la retención de humedad edáfica por manejo de residuos.
* **Heterogeneidad Intra-Parcelaria y Geometría (`landsat_intra_vi6t_std`, `planet_msavi_mean`, `elongacion_bbox`):** Cuantifican la variación térmica dentro del polígono, el vigor ajustado al brillo del suelo desnudo a $3\text{ m}$ y la forma de las melgas agrícolas.

---

## Arquitectura de Modelos por Entidad Federativa

Debido a la divergencia biofísica entre estados, el pipeline entrena tres especialistas independientes validados mediante **Validación Cruzada Out-Of-Fold ($K=3$, `SEED=42`)**, ajustando escaladores, imputadores y priores estrictamente dentro de cada fold para evitar fuga de información (*data leakage*):

### 1. Especialista Hidalgo ($N=36 \to r = 0.9153, R^2 = 0.8364, \text{MAE} = 227.7\text{ kg/ha}$)
* **Régimen:** Fuerte gradiente hidro-térmico y productivo entre valles húmedos tecnificados (Almoloya, Apan, Cuautepec) y laderas secas (Singuilucan, Tepeapulco).
* **Arquitectura:** Ensamble Híbrido **75% `ExtraTreesRegressor`** (`n_estimators=130, max_depth=5`) con **Target Encoding Bayesiano Suavizado ($m=0.5$)** del municipio + **25% `SVR` (Kernel RBF)**.
* **Variables clave:** Estrés termo-hídrico (`landsat_vi6t_max * s2_ndwi_min`), absorción de clorofila en borde rojo (`s2_mra_mean`), anomalías térmicas de siembra con *downscaling* adiabático (`lp_Tmin_anom25_04`), variabilidad de déficit de presión de vapor (`nn_vpd_vege_std`) y diferencial térmico dosel-aire (`bi_canopy_air_diff`).

### 2. Especialista Puebla ($N=71 \to r = 0.4410, R^2 = 0.1691, \text{MAE} = 361.1\text{ kg/ha}$)
* **Régimen:** Cuenca húmeda de alta productividad (Chignahuapan) con **saturación severa del dosel óptico ($\text{NDVI} > 0.70$)** y presencia de **valores atípicos extremos ($2.15\text{ t/ha}$)** por siniestros climáticos focalizados (heladas o acame).
* **Arquitectura:** Fusión 50/50 entre dos ramas de ensamble no lineal robusto (**45% `SVR RBF` ($\epsilon=0.15$) + 35% `HuberRegressor` ($\epsilon=1.2$) + 15% `GaussianProcessRegressor` (Kernel RBF + WhiteKernel) + 5% `ExtraTrees` (`max_depth=2`)**):
  * **Rama A (50%):** Opera directamente sobre topografía, rugosidad local (`proxy_relieve_m`), labranza de conservación pre-siembra (`s2_sti_presow`) y estrés aero-hídrico (`bi_vpd_ndwi_stress`).
  * **Rama B (50%):** Incorpora **2 componentes latentes supervisados (`PLSRegression(n_components=2)`)** extraídos de la matriz climática CHIRTS-ERA5, temperaturas cardinales, sincronización fenológica e índices no saturables (`FAPAR`, `SAVI`, `MRA`).

### 3. Especialista Tlaxcala ($N=31 \to r = 0.3821, R^2 = 0.0552, \text{MAE} = 274.8\text{ kg/ha}$)
* **Régimen:** Rendimientos reportados en campo en **escalones discretos ($\{2.5, 3.0, 3.5\}\text{ t/ha}$)** con baja varianza muestral en Calpulalpan y Nanacamilpa.
* **Arquitectura:** **45% Clasificador Ordinal Probabilístico `SVC` (`kernel='rbf', C=0.5, probability=True`)** que estima el valor esperado continuo $\mathbb{E}[Y] = \sum P(c_k)\cdot c_k$ + **55% Anclaje a la Mediana Regional ($3.0\text{ t/ha}$)**.
* **Variables clave:** Vigor ajustado al suelo a $3\text{ m}$ (`planet_msavi_mean`), heterogeneidad térmica intra-parcelaria (`landsat_intra_vi6t_std`), elongación geométrica del polígono (`elongacion_bbox`) y coordenadas geográficas.

---


