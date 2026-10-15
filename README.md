# Pipeline Oficial AgroRaíces — Metodología CRISP-DM + Arquitectura QBAF / DF-QuAD + SHAP

### Reto AgroCebada FIRA 2026 | Estimación de Rendimiento de Cebada Maltera (t/ha) en Hidalgo, Puebla y Tlaxcala

---

**Equipo:** TecnoBuhos  
**Modelo de Proceso:** CRISP-DM (*Cross-Industry Standard Process for Data Mining*)  
**Arquitectura de Modelado:** Marco Neuro-Simbólico de Argumentación Bipolar Cuantitativa (**QBAF**) + Algoritmo **DF-QuAD** con Validación *Undercutting* y Doble Separación (por Estado y Grupos Homogéneos) + IA Explicable (**SHAP**)  
**Cuaderno Principal:** [`pipeline_agroraices.ipynb`](pipeline_agroraices.ipynb)  
**Predicciones Oficiales:** [`predicciones_oficiales_agroraices.csv`](predicciones_oficiales_agroraices.csv)  
**Modelo Empaquetado para Producción:** [`modelo_agroraices_final.joblib`](modelo_agroraices_final.joblib) / [`modelo_metadata.json`](modelo_metadata.json)

---

## Marco Metodológico CRISP-DM en AgroRaíces

El proyecto **AgroRaíces** adopta como marco directriz el estándar internacional **CRISP-DM** (*Cross-Industry Standard Process for Data Mining*) para organizar la ingeniería de características satelitales, el modelado neuro-simbólico y la inferencia predictiva explicable a escala de lote agrícola.

El flujo de trabajo se estructura en **seis fases interconectadas**:

| Fase CRISP-DM | Pregunta central | Resultado esperado |
| :--- | :--- | :--- |
| **1. Comprensión del negocio** | ¿Qué problema se quiere resolver y cómo se reconocerá el éxito? | Objetivos, restricciones y criterios de éxito. |
| **2. Comprensión de los datos** | ¿Qué datos existen y qué tan adecuados son? | Inventario, exploración y diagnóstico de calidad. |
| **3. Preparación de los datos** | ¿Cómo se transformarán los datos para modelar? | Base analítica integrada y estructurada en Grupos Homogéneos. |
| **4. Modelado** | ¿Qué modelos pueden resolver el problema? | Arquitectura Neuro-Simbólica **QBAF + DF-QuAD** entrenada y validada por invariancia. |
| **5. Evaluación** | ¿El modelo cumple el objetivo y generaliza? | Modelo seleccionado ($r = 0.8584$, $R^2 = 0.7362$, $\text{MAE} = 296.3\text{ kg/ha}$), métricas y análisis de errores. |
| **6. Despliegue y Explicabilidad** | ¿Cómo se utilizarán, auditarán y comunicarán los resultados? | Predicciones oficiales, descomposición aditiva **SHAP** por Estado (3 ejemplos por Estado) y empaquetado web. |

> [!IMPORTANT]
> **Principio de No Linealidad y Retroalimentación hacia la Arquitectura QBAF + DF-QuAD:**  
> En concordancia con CRISP-DM, el pipeline no representa una secuencia lineal rígida sino un ciclo con retroalimentación empírica activa:
> * **Evaluación $\leftrightarrow$ Preparación de Datos (Capa 0 de *Undercutting*):** La evaluación preliminar reveló que los modelos puramente ópticos colapsaban en Puebla debido a la saturación asintótica de reflectancia foliar ($\text{NDVI} > P_{75}$). Esto retroalimentó la preparación de datos para extraer índices de labranza pre-siembra (`s2_sti_presow`, `s2_ndti_presow`), humedad `SWIR` no saturable (`s2_dswi2_mean`) e introducir una **Capa 0 de validación neuro-simbólica (*Ataque Undercutting*)** que anula el voto del árbol óptico ($\sigma_{\text{ET}} \to 0.001$) cuando el dosel se satura.
> * **Evaluación $\leftrightarrow$ Modelado (Debate DF-QuAD y Selección de Nodos por Invariancia):** La evaluación mediante validación cruzada ($K=3$), ruido sintético gaussiano ($20\%$) e invariancia ante un $20\%$ de nubosidad demostró que cada **Grupo Homogéneo de Variables (Nodo)** requiere una geometría algorítmica específica (`GradientBoostingRegressor` en el nodo biótico de Hidalgo, `SVR RBF`/`GPR` en el nodo de labranza de Puebla, `HuberRegressor` en el nodo termo-hídrico SWIR, `PLS(k=2)` en el nodo bioclimático cardinal y `SVC Ordinal Probabilístico` + Anclaje Modal en el régimen discretizado de Tlaxcala), arbitrados formalmente mediante el algoritmo **DF-QuAD**.

---

## 1. Fase 1: Comprensión del Negocio

* **Pregunta central:** *¿Qué problema se quiere resolver y cómo se reconocerá el éxito?*
* **Resultado esperado:** *Objetivos, restricciones y criterios de éxito.*

### 1.1 Contexto, Definición del Problema y Enfoque Neuro-Simbólico (QBAF + DF-QuAD)
El **Reto AgroCebada FIRA 2026** tiene como objetivo estimar con alta precisión agronómica el rendimiento de cosecha de cebada maltera (*Hordeum vulgare*, en $\text{t/ha}$) a nivel de polígono predial individual en **Hidalgo**, **Puebla** y **Tlaxcala**.

En la literatura de teledetección (*Lobell et al., 2015; Deines et al., 2021*) y de inteligencia artificial explicable (*Rago, Cocarascu & Toni*), este desafío se resuelve mediante una **Arquitectura Neuro-Simbólica de Argumentación Bipolar Cuantitativa (QBAF) y Debates DF-QuAD** basada en una **Doble Separación**:
1. **Separación en Grupos Homogéneos de Variables (Nodos de Evidencia):** Las covariables se agrupan en 5 bloques físicamente homogéneos ($G_{\text{Bio}}$, $G_{\text{Geo-Labranza}}$, $G_{\text{SWIR-Termo}}$, $G_{\text{Clima-PLS}}$, $G_{\text{Meseta-Modal}}$).
2. **Separación por Estado y Grupos Homogéneos de Parcelas (Meta-Árbitro Contextual):**
   * **Hidalgo (`Hidalgo_G1_Valle_Humedo` vs. `Hidalgo_G2_Ladera_Seca`):** El árbitro DF-QuAD otorga soporte prioritario ($\sigma = 0.75$) al nodo biótico (`GradientBoosting` + Prior Bayesiano municipal) y modera ($\sigma = 0.25$) al nodo topográfico (`SVR RBF`).
   * **Puebla (`Puebla_G1_Conservacion_Labranza` vs. `Puebla_G2_Dosel_Saturado`):** El árbitro aplica un **ataque *Undercutting*** al sensor óptico cuando $\text{NDVI} > P_{75}$ ($\sigma_{\text{ET}} = 0.001$) y transfiere el soporte a labranza de conservación (`SVR`), humedad SWIR (`Huber`) y clima fenológico (`PLS k=2`).
   * **Tlaxcala (`Tlaxcala_G1_Meseta_Modal`):** Régimen de meseta con rendimientos reportados en escalones discretos ($2.5, 3.0, 3.5\text{ t/ha}$), donde el árbitro pondera el Anclaje Modal ($\sigma = 0.55$) con clasificación ordinal probabilística `SVC` ($\sigma = 0.45$).

### 1.2 Criterios de Éxito
* **Criterio Cuantitativo Nacional:** $r \ge 0.85$, $R^2 \ge 0.72$ y $\text{MAE} \le 310\text{ kg/ha}$ en Validación Cruzada Out-Of-Fold ($K=3$) sin fuga de información.
* **Criterio Regional:** Correlaciones positivas y estadísticamente significativas en todos los estados ($r > 0.90$ en Hidalgo, $r > 0.40$ en Puebla y $r > 0.35$ en Tlaxcala).
* **Criterio Operativo y de Explicabilidad:** Generación íntegra de las 59 predicciones de prueba ciega dentro del rango biológico $[1.50, 4.80]\text{ t/ha}$, auditoría aditiva mediante **SHAP** por Estado y serialización para producción web.

---

## 2. Fase 2: Comprensión de los Datos

* **Pregunta central:** *¿Qué datos existen y qué tan adecuados son?*
* **Resultado esperado:** *Inventario, exploración y diagnóstico de calidad.*

| Fuente | Tipo de Información | Temporalidad / Resolución | Registros | Indicadores Clave |
| :--- | :--- | :--- | :---: | :--- |
| **`Conjunto_datos_BASICO_AgroCebada2026.csv`** | Reflectancia multiespectral y térmica superficial | 2022–2025 ($10\text{ m}$–$30\text{ m}$) | $107,666$ | Sentinel-2 (`NDWI`, `DSWI2`, `FAPAR`, `SAVI`, `MRA`, `STI`, `NDTI`) y Landsat 8/9 (`VI6T`). |
| **`Conjunto_datos_PRO_AgroCebada.csv`** | Alta resolución óptica PlanetScope | 2025 ($3\text{ m}$) | $47,804$ | Dinámica foliar fina (`LAI`, `NDVI`, `MSAVI`) y heterogeneidad intra-parcelaria (`ndvi_std`). |
| **`temperatura_chirts_era5_parcelas.csv`** | Cubo climático híbrido CHIRTS-ERA5 | 2022–2025 ($0.05^\circ \approx 5\text{ km}$) | 96 GeoTIFF | Temperaturas mínimas y máximas mensuales, anomalías climáticas y tiempo térmico cardinal. |
| **`topografia_inegi_cem4_parcelas.csv`** | Fisiografía Continuo de Elevaciones CEM 4.0 | Estático | $197$ | Altitud sobre el nivel del mar (`elevacion_msnm`) y pendiente topográfica (`pendiente_grados`). |
| **`parcelas_con_topografia.csv`** | Micro-rugosidad de cuenca | Estático | $197$ | Índice de relieve local (`proxy_relieve_m`). |
| **`shp/Parcelas_Reto_AGC_CONJUNTO.shp`** | Polígonos vectoriales oficiales | UTM Zona 14N (`EPSG:32614`) | $197$ | Delimitación predial, centroides geográficos y morfometría (`elongacion_bbox`). |
| **`ID_area_rendimiento_70_30_Reto_AgroCebada.csv`** | Ground truth oficial | Ciclo evaluado | $197$ | Superficie (`AREA_HA`) y rendimiento cosechado (`138` Entrenamiento / `59` Prueba Ciega). |

---

## 3. Fase 3: Preparación de los Datos

* **Pregunta central:** *¿Cómo se transformarán los datos para modelar?*
* **Resultado esperado:** *Base analítica integrada y estructurada en Grupos Homogéneos para el marco QBAF + DF-QuAD.*

1. **Capa 0 de Validación Física (*Undercutting* Radiométrico):** Descarte estricto de escenas ópticas con $\text{nubosidad} > 30\%$ para impedir depresiones artificiales de reflectancia antes de construir los argumentos base.
2. **Nodo Fisiográfico y Morfométrico ($G_{\text{Geo-Labranza}}$):** Proyección UTM Zona 14N (`EPSG:32614`), centroides (`lat_centro`, `lon_centro`), elongación (`elongacion_bbox`), elevación y pendiente del CEM 4.0 de INEGI e índices de labranza y cobertura de rastrojo en pre-siembra (`s2_sti_presow`, `s2_ndti_presow`).
3. **Nodos Biofísicos Multitemporales por Sensor ($G_{\text{Bio}}$ y $G_{\text{SWIR-Termo}}$):** Retención hídrica foliar (`s2_ndwi_mean`, `s2_ndwi_min`, `s2_dswi2_mean` en banda SWIR inmune a saturación óptica), absorción en borde rojo (`s2_mra_mean`), emisión térmica Landsat (`landsat_vi6t_max`, `landsat_vi6t_std`) y vigor fino PlanetScope a $3\text{ m}$ (`planet_lai_std`, `planet_msavi_mean`, `planet_ndvi_mean`, `persistencia_verde_m11`).
4. **Nodo Bioclimático Cardinal Fenológico ($G_{\text{Clima-PLS}}$):** Corrección adiabática ($\Gamma = -6.5^\circ\text{C}/\text{km}$) sobre las 96 capas CHIRTS-ERA5, anomalías térmicas (`bi_Tmin_anom25_04`, `lp_Tmin_anom25_04`) y acoplamiento termo-hídrico (`estres_termohidrico`).

---

## 4. Fase 4: Modelado (Arquitectura QBAF + DF-QuAD)

* **Pregunta central:** *¿Qué modelos pueden resolver el problema?*
* **Resultado esperado:** *Arquitectura Neuro-Simbólica QBAF + DF-QuAD entrenada y validada por invariancia.*

### 4.1 Arquitectura de Árbitros Regionales QBAF y Deliberación DF-QuAD

```
                     Matriz Analítica Integrada (197 parcelas: 138 Train / 59 Test)
                                                    │
                            [Capa 0: Validación Física y Ataques Undercutting]
                                                    │
                 ┌──────────────────────────────────┼──────────────────────────────────┐
                 ▼                                  ▼                                  ▼
      Árbitro QBAF Hidalgo                Árbitro QBAF Puebla               Árbitro QBAF Tlaxcala
      (N = 36 | Nodo 1: G_Bio)           (N = 71 | Nodos 2, 3 y 4)         (N = 31 | Nodo 5: G_Meseta)
                 │                                  │                                  │
      ┌──────────┴──────────┐            ┌──────────┴──────────┐            ┌──────────┴──────────┐
      ▼                     ▼            ▼                     ▼            ▼                     ▼
 σ = 0.75 Soporte     σ = 0.25 Mod.  Undercutting Óptico   Debate DF-QuAD  σ = 0.45 SVC         σ = 0.55 Anclaje
 GradientBoosting +   SVR Kernel RBF si NDVI > P75         SVR + Huber +   Ordinal Prob.        Modal Regional
 Prior Bayes (m=0.5)  (Topografía)   (σ_ET -> 0.001)       GPR + PLS(k=2)  (Clases 0.5 t/ha)    (3.0 t/ha)
```

### 4.1.1 Justificación Cuantitativa de la Selección de Algoritmos por Nodo ($K=3$ CV, Ruido Sintético e Invariancia a 20% de Nubes)

| Nodo QBAF (Grupo Homogéneo) | Modelo Seleccionado (Ganador) | Métricas del Modelo Ganador ($K=3$ OOF) | Métricas de Otros Modelos Evaluados (Rivales en el Nodo) | Criterio de Elección Neuro-Simbólica |
| :--- | :--- | :--- | :--- | :--- |
| **Nodo 1 ($G_{\text{Bio}}$ — Hidalgo)**<br>*Biótico, Borde Rojo e Hídrico* | **`GradientBoostingRegressor`**<br>*(+ Prior Bayesiano $m=0.5$)* | • **$R^2_{\text{limpio}} = 0.8896$**<br>• **$\text{MAE}_{\text{limpio}} = 192.20\text{ kg/ha}$**<br>• $\text{MAE}_{\text{ruido 20\%}} = 231.03\text{ kg/ha}$<br>• **$\text{MAE}_{\text{nubes 20\%}} = 243.99\text{ kg/ha}$** | • `RandomForest`: $R^2=0.8717$, $\text{MAE}=216.43$, Ruido $=244.43$, Nubes $=246.38\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=0.8211$, $\text{MAE}=243.68$, Ruido $=282.71$, Nubes $=278.46\text{ kg/ha}$<br>• `Ridge`: $R^2=0.7859$, $\text{MAE}=288.90$, Ruido $=333.76$, Nubes $=1,044.98\text{ kg/ha}$<br>• `Huber`: $R^2=0.6734$, $\text{MAE}=330.41$, Ruido $=391.77$, Nubes $=962.61\text{ kg/ha}$<br>• `SVR RBF`: $R^2=0.6029$, $\text{MAE}=316.20$, Ruido $=344.20$, Nubes $=646.70\text{ kg/ha}$ | Mayor precisión en limpio y máxima invariancia ante depresión radiométrica por nubosidad ($20\%$), donde modelos lineales (`Ridge`, `Huber`) colapsan. |
| **Nodo 2 ($G_{\text{Geo-Labranza}}$ — Puebla)**<br>*Fisiografía y Labranza Pre-Siembra* | **`SVR (Kernel RBF)` + `GPR (RBF + WhiteKernel)`** | • **`SVR`**: **$R^2_{\text{limpio}} = 0.0910$**, **$\text{MAE}_{\text{limpio}} = 385.70$**, Ruido $=387.69$, Nubes $=411.35\text{ kg/ha}$<br>• **`GPR`**: $R^2_{\text{limpio}} = 0.0568$, $\text{MAE}_{\text{limpio}} = 399.83$, **$\text{MAE}_{\text{nubes 20\%}} = 388.00\text{ kg/ha}$** | • `Ridge`: $R^2=0.0364$, $\text{MAE}=410.99$, Ruido $=418.74$, Nubes $=740.93\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=0.0043$, $\text{MAE}=415.95$, Ruido $=424.67$, Nubes $=420.23\text{ kg/ha}$<br>• `Huber`: $R^2=-0.0170$, $\text{MAE}=394.61$, Ruido $=409.59$, Nubes $=706.59\text{ kg/ha}$<br>• `RandomForest`: $R^2=-0.1994$, $\text{MAE}=473.40$, Nubes $=475.15\text{ kg/ha}$<br>• `GradientBoosting`: $R^2=-0.2442$, $\text{MAE}=501.90$, Nubes $=493.89\text{ kg/ha}$ | `SVR RBF` lidera en precisión en limpio y ruido gaussiano, mientras `GPR` aporta la menor degradación bajo $20\%$ de perturbación superficial. |
| **Nodo 3 ($G_{\text{SWIR-Termo}}$ — Puebla)**<br>*Humedad SWIR y Emisión Térmica* | **`HuberRegressor` ($\epsilon = 1.2$)** | • **$R^2_{\text{limpio}} = 0.0664$**<br>• **$\text{MAE}_{\text{limpio}} = 365.56\text{ kg/ha}$**<br>• **$\text{MAE}_{\text{ruido 20\%}} = 384.26\text{ kg/ha}$**<br>• **$\text{MAE}_{\text{nubes 20\%}} = 361.46\text{ kg/ha}$** | • `Ridge`: $R^2=0.0571$, $\text{MAE}=397.97$, Ruido $=412.11$, Nubes $=526.74\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=0.0272$, $\text{MAE}=403.53$, Ruido $=412.50$, Nubes $=420.51\text{ kg/ha}$<br>• `SVR RBF`: $R^2=-0.0068$, $\text{MAE}=402.07$, Ruido $=406.04$, Nubes $=476.33\text{ kg/ha}$<br>• `GPR`: $R^2=-0.0169$, $\text{MAE}=416.29$, Nubes $=429.40\text{ kg/ha}$<br>• `RandomForest`: $R^2=-0.1103$, $\text{MAE}=444.59\text{ kg/ha}$<br>• `GradientBoosting`: $R^2=-0.3312$, $\text{MAE}=492.37\text{ kg/ha}$ | Su función de pérdida híbrida $L_2/L_1$ acota linealmente los valores atípicos térmicos (`vi6t_std`) producidos por sombras de nubes y heladas. |
| **Nodo 4 ($G_{\text{Clima-PLS}}$ — Puebla)**<br>*Cubo Climático Cardinal Fenológico* | **`PLSRegression (k=2)` acoplado a Especialistas** | • **$R^2_{\text{limpio}} = 0.1103$**<br>• **$\text{MAE}_{\text{limpio}} = 403.70\text{ kg/ha}$**<br>• **$\text{MAE}_{\text{ruido 20\%}} = 401.29\text{ kg/ha}$**<br>• **$\text{MAE}_{\text{nubes 20\%}} = 410.72\text{ kg/ha}$** | • `PCA No Supervisado (k=2) + SVR`: $R^2=0.0378$, $\text{MAE}=430.69$, Nubes $=435.46\text{ kg/ha}$<br>• `Sin Compresión (SVR Directo)`: $R^2=0.0198$, $\text{MAE}=405.77$, Nubes $=433.63\text{ kg/ha}$<br>• `Sin Compresión (ExtraTrees)`: $R^2=-0.1710$, $\text{MAE}=472.48$, Nubes $=482.58\text{ kg/ha}$ | Comprime la multicolinealidad severa ($\kappa(\mathbf{X}) = 381.9$) maximizando la covarianza supervisada con el rendimiento antes del debate DF-QuAD. |
| **Nodo 5 ($G_{\text{Meseta-Modal}}$ — Tlaxcala)**<br>*Meseta de Rendimiento Discretizado* | **`SVC Ordinal Probabilístico` + Anclaje Modal ($3.0\text{ t/ha}$)** | • **$R^2_{\text{limpio}} = 0.0552$**<br>• **$\text{MAE}_{\text{limpio}} = 274.77\text{ kg/ha}$**<br>• **$\text{MAE}_{\text{ruido 20\%}} = 280.17\text{ kg/ha}$**<br>• **$\text{MAE}_{\text{nubes 20\%}} = 298.32\text{ kg/ha}$** | • `SVR Continuo`: $R^2=0.0888$, $\text{MAE}=274.60$, pero **colapsa con nubes a $380.46\text{ kg/ha}$ ($+105.86\text{ kg/ha}$)**<br>• `GPR`: $R^2=-0.0029$, $\text{MAE}=309.51$, Nubes $=309.51\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=-0.0434$, $\text{MAE}=306.30$, Nubes $=324.47\text{ kg/ha}$<br>• `RandomForest`: $R^2=-0.1068$, $\text{MAE}=313.05$, Nubes $=332.98\text{ kg/ha}$<br>• `Ridge`: $R^2=-0.2757$, $\text{MAE}=356.28$, Nubes $=625.81\text{ kg/ha}$<br>• `Huber`: $R^2=-0.4905$, $\text{MAE}=378.63$, Nubes $=686.56\text{ kg/ha}$<br>• `GradientBoosting`: $R^2=-0.7121$, $\text{MAE}=361.10\text{ kg/ha}$ | Evita el sobreajuste sobre etiquetas discretizadas ($80.6\%$ en múltiplos de $0.5\text{ t/ha}$) y resiste el $20\%$ de nubosidad donde `SVR Continuo` colapsa. |

---

## 5. Fase 5: Evaluación

* **Pregunta central:** *¿El modelo cumple el objetivo y generaliza?*
* **Resultado esperado:** *Modelo seleccionado, métricas y análisis de errores.*

### 5.1 Desempeño Oficial en Validación Cruzada ($K=3$ Out-Of-Fold)

| Ámbito / Árbitro QBAF | Parcelas ($N$) | Correlación de Pearson ($r$) | Coeficiente de Determinación ($R^2$) | $\text{MAE}$ ($\text{t/ha}$) | $\text{MAE}$ ($\text{kg/ha}$) | $\text{RMSE}$ ($\text{kg/ha}$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Nacional (Global QBAF + DF-QuAD)** | **$138$** | **$0.8584$** | **$0.7362$** | **$0.2963$** | **$296.3\text{ kg/ha}$** | **$435.6\text{ kg/ha}$** |
| **Árbitro QBAF Hidalgo** | $36$ | **$0.9436$** | **$0.8885$** | **$0.1882$** | **$188.2\text{ kg/ha}$** | **$256.0\text{ kg/ha}$** |
| **Árbitro QBAF Puebla** | $71$ | **$0.4433$** | **$0.1710$** | **$0.3605$** | **$360.5\text{ kg/ha}$** | **$528.3\text{ kg/ha}$** |
| **Árbitro QBAF Tlaxcala** | $31$ | **$0.3821$** | **$0.0552$** | **$0.2748$** | **$274.8\text{ kg/ha}$** | **$359.6\text{ kg/ha}$** |

---

## 6. Fase 6: Despliegue y Explicabilidad con SHAP + DF-QuAD por Estado

* **Pregunta central:** *¿Cómo se utilizarán, auditarán y comunicarán los resultados?*
* **Resultado esperado:** *Predicciones oficiales, auditoría aditiva SHAP sobre 3 ejemplos sin etiqueta por Estado y empaquetado web.*

### 6.1 Predicciones Auditadas con SHAP por Estado (3 Ejemplos de Prueba Ciega por Estado = 9 Parcelas)

Cada predicción sobre los polígonos del conjunto `CONJUNTO == 'PREDICCION'` se audita mediante la descomposición aditiva exacta de **SHAP (`shap.Explainer`)**:
$$\hat{y}_{\text{SHAP}}(x) = \phi_0 + \sum_{j=1}^{M} \phi_j(x)$$

| Estado | `ID_POLIGONO` | Municipio | Área ($\text{ha}$) | Deliberación QBAF + DF-QuAD | Base Regional $\phi_0$ ($\text{t/ha}$) | Suma SHAP $\sum \phi_j$ ($\text{kg/ha}$) | Predicción $\hat{y}$ ($\text{t/ha}$) | Producción Total ($\text{t}$) | Top Impulsores ($+\text{SHAP}$) | Top Frenadores ($-\text{SHAP}$) |
| :--- | :---: | :--- | :---: | :--- | :---: | :---: | :---: | :---: | :--- | :--- |
| **Hidalgo** | **`AGC_003`** | Singuilucan | $18.68$ | Nodo 1 `GradientBoosting` ($\sigma=0.75$) + `SVR RBF` ($\sigma=0.25$) | $3.7669$ | $-379.3$ | **$3.3876$** | **$63.28$** | `s2_mra_mean` ($+117\text{ kg/ha}$), `bi_canopy_air_diff` ($+21\text{ kg/ha}$) | `bi_Tmin_anom25_04` ($-211\text{ kg/ha}$), `prior_municipal` ($-136\text{ kg/ha}$) |
| **Hidalgo** | **`AGC_030`** | Singuilucan | $13.05$ | Nodo 1 `GradientBoosting` ($\sigma=0.75$) + `SVR RBF` ($\sigma=0.25$) | $3.7669$ | $-646.2$ | **$3.1207$** | **$40.73$** | `s2_mra_mean` ($+39\text{ kg/ha}$), `landsat_vi6t_max` ($+11\text{ kg/ha}$) | `bi_Tmin_anom25_04` ($-227\text{ kg/ha}$), `prior_municipal` ($-152\text{ kg/ha}$) |
| **Hidalgo** | **`AGC_042`** | Singuilucan | $3.81$ | Nodo 1 `GradientBoosting` ($\sigma=0.75$) + `SVR RBF` ($\sigma=0.25$) | $3.7669$ | $-1,022.0$ | **$2.7448$** | **$10.46$** | `s2_mra_mean` ($+17\text{ kg/ha}$), `landsat_vi6t_max` ($+12\text{ kg/ha}$) | `bi_Tmin_anom25_04` ($-422\text{ kg/ha}$), `lp_Tmin_anom25_04` ($-293\text{ kg/ha}$) |
| **Puebla** | **`AGC_011`** | Chignahuapan | $7.17$ | *Undercutting* Activo ($\sigma_{\text{ET}}=0.001$) + `SVR`/`Huber`/`PLS` | $4.5346$ | $+173.0$ | **$4.7076$** | **$33.75$** | `landsat_vi6t_std` ($+49\text{ kg/ha}$), `Z1_pls_clima` ($+44\text{ kg/ha}$) | `log_area` ($-53\text{ kg/ha}$), `s2_ndwi_min` ($-24\text{ kg/ha}$) |
| **Puebla** | **`AGC_016`** | Chignahuapan | $7.47$ | Debate Completo `SVR`/`Huber`/`GPR`/`ET` + `PLS` | $4.5346$ | $-118.3$ | **$4.4164$** | **$32.99$** | `bi_vpd_ndwi_stress` ($+14\text{ kg/ha}$), `s2_ndwi_min` ($+10\text{ kg/ha}$) | `s2_sti_presow` ($-68\text{ kg/ha}$), `log_area` ($-66\text{ kg/ha}$) |
| **Puebla** | **`AGC_028`** | Chignahuapan | $1.86$ | Debate Completo `SVR`/`Huber`/`GPR`/`ET` + `PLS` | $4.5346$ | $+120.0$ | **$4.6546$** | **$8.66$** | `s2_ndwi_min` ($+105\text{ kg/ha}$), `log_area` ($+59\text{ kg/ha}$) | `bi_vpd_ndwi_stress` ($-53\text{ kg/ha}$), `s2_dswi2_mean` ($-50\text{ kg/ha}$) |
| **Tlaxcala** | **`AGC_025`** | Calpulalpan | $2.00$ | Nodo 5 Anclaje Modal ($\sigma=0.55$) + `SVC` Ordinal ($\sigma=0.45$) | $2.9941$ | $+12.4$ | **$3.0065$** | **$6.01$** | `elongacion_bbox` ($+6\text{ kg/ha}$), `planet_msavi_mean` ($+4\text{ kg/ha}$) | `lat_centro` ($-2\text{ kg/ha}$) |
| **Tlaxcala** | **`AGC_027`** | Calpulalpan | $8.46$ | Nodo 5 Anclaje Modal ($\sigma=0.55$) + `SVC` Ordinal ($\sigma=0.45$) | $2.9941$ | $-13.6$ | **$2.9805$** | **$25.21$** | `lat_centro` ($+3\text{ kg/ha}$), `elongacion_bbox` ($+1\text{ kg/ha}$) | `planet_msavi_mean` ($-12\text{ kg/ha}$), `landsat_intra_vi6t_std` ($-5\text{ kg/ha}$) |
| **Tlaxcala** | **`AGC_029`** | Calpulalpan | $5.90$ | Nodo 5 Anclaje Modal ($\sigma=0.55$) + `SVC` Ordinal ($\sigma=0.45$) | $2.9941$ | $+26.5$ | **$3.0206$** | **$17.82$** | `planet_msavi_mean` ($+8\text{ kg/ha}$), `lon_centro` ($+8\text{ kg/ha}$) | Sin castigo neto |

### 6.2 Empaquetado y Serialización para Producción Web
El script de producción [`empaquetar_modelo_agroraices_final.py`](empaquetar_modelo_agroraices_final.py) entrena y serializa la arquitectura completa **QBAF + DF-QuAD**:
1. [`modelo_agroraices_final.joblib`](modelo_agroraices_final.joblib): Artefacto binario comprimido ($683.3\text{ KB}$) con todos los nodos de evidencia, escaladores, priores bayesianos y umbrales de *Undercutting*.
2. [`modelo_metadata.json`](modelo_metadata.json): Esquema JSON con las métricas oficiales Out-Of-Fold ($r = 0.8584, R^2 = 0.7362, \text{MAE} = 296.3\text{ kg/ha}$) y municipios soportados.
3. [`predicciones_oficiales_agroraices.csv`](predicciones_oficiales_agroraices.csv): Archivo oficial con las estimaciones de rendimiento ($\text{t/ha}$) y producción total ($\text{t}$) para las 59 parcelas de prueba ciega.

---

## Estructura del Repositorio

```
AgroRaíces/
├── pipeline_agroraices.ipynb             # Pipeline oficial CRISP-DM + QBAF/DF-QuAD + SHAP por Estado
├── predicciones_oficiales_agroraices.csv  # Estimaciones oficiales para las 59 parcelas ciegas
├── empaquetar_modelo_agroraices_final.py # Script de entrenamiento y empaquetado para producción web
├── modelo_agroraices_final.joblib        # Modelo QBAF + DF-QuAD serializado para producción
├── modelo_metadata.json                  # Metadatos y métricas oficiales del modelo en JSON
├── diagnostico_regional_agroraices.png   # Panel visual regional (Hidalgo, Puebla, Tlaxcala)
├── diagnostico_global_agroraices.png     # Panel visual global (1:1, residuos y densidades)
├── shap_prediccion_3x3_por_estado.png    # Panel 3x3 de explicabilidad SHAP en prueba ciega por Estado
├── README.md                             # Documentación oficial del proyecto AgroRaíces
└── Conjunto_datos_*.csv                  # Tablas de datos oficiales del reto FIRA 2026
```
