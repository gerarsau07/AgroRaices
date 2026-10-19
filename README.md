# Reto AgroCebada FIRA 2026 | Estimación de Rendimiento de Cebada  (t/ha) en Hidalgo, Puebla y Tlaxcala

---

**Equipo:** TecnoBuhos  
**Modelo de Proceso:** CRISP-DM (*Cross-Industry Standard Process for Data Mining*)  
**Arquitectura de Modelado:** Marco Neuro-Simbólico de Argumentación Bipolar Cuantitativa (**QBAF**) + Algoritmo **DF-QuAD** + IA Explicable (**XAI / SHAP**)  
**Cuaderno Principal (Pipeline Oficial):** [`notebooks/04_pipeline_agroraices.ipynb`](notebooks/04_pipeline_agroraices.ipynb)  
**Cuadernos de Experimentación del Modelo:** [`notebooks/03_seleccion_modelos_por_nodo.ipynb`](notebooks/03_seleccion_modelos_por_nodo.ipynb) | [`notebooks/02_SeleccionArquitectura.ipynb`](notebooks/02_SeleccionArquitectura.ipynb) | [`notebooks/01_mapas.ipynb`](notebooks/01_mapas.ipynb)  
**Prototipo Web Desplegado (Agricultor y Analista):** [https://agro-raices-web.vercel.app/](https://agro-raices-web.vercel.app/)  

---

## Arquitectura del Proyecto y Estructura de Carpetas

El eje central del proyecto es la **Metodología CRISP-DM** y el cuaderno principal **[`notebooks/04_pipeline_agroraices.ipynb`](notebooks/04_pipeline_agroraices.ipynb)**, donde se desarrolla de principio a fin el pipeline oficial. Los demás cuadernos en **`notebooks/`** son fundamentales como **laboratorio de experimentación** para sustentar las decisiones del modelo:

```text
AgroRaíces/
├── notebooks/                                             # Cuadernos científicos ejecutables
│   ├── 01_mapas.ipynb                                     # [Experimentación] Análisis y visualización geoespacial de las parcelas
│   ├── 02_SeleccionArquitectura.ipynb                     # [Experimentación] Estudio de opciones de modelos
│   ├── 03_seleccion_modelos_por_nodo.ipynb                # [Experimentación] Naturaleza matemática de los datos y torneo por Nodo Homogéneo
│   └── 04_pipeline_agroraices.ipynb                       # CUADERNO PRINCIPAL: Pipeline oficial CRISP-DM + QBAF/DF-QuAD + XAI (SHAP)
└── README.md                                              # Documentación integral del proyecto
```

---

## Marco Metodológico CRISP-DM en AgroRaíces

El proyecto **AgroRaíces** adopta el estándar internacional **CRISP-DM** (*Cross-Industry Standard Process for Data Mining*) integrado con una arquitectura neuro-simbólica en seis fases interconectadas:

| Fase CRISP-DM | Pregunta central | Resultado esperado y Cuaderno Asociado |
| :--- | :--- | :--- |
| **1. Comprensión del negocio** | ¿Qué problema se quiere resolver y cómo se reconocerá el éxito? | Objetivos, restricciones y enfoque de Doble Separación (`04_pipeline_agroraices.ipynb`). |
| **2. Comprensión de los datos** | ¿Qué datos existen y qué tan adecuados son? | Inventario de `data/`, exploración geoespacial (`01_mapas.ipynb`) y control radiométrico ($\le 30\%$ nubes). |
| **3. Preparación de los datos** | ¿Cómo se transformarán los datos para modelar? | Capa 0 de *Undercutting* y estructuración en 5 Nodos Homogéneos (`04_pipeline_agroraices.ipynb`). |
| **4. Modelado** | ¿Qué modelos pueden resolver el problema y por qué? | Deliberación **QBAF + DF-QuAD** en el pipeline principal (`04_pipeline_agroraices.ipynb`), respaldada por la experimentación por nodo (`03_seleccion_modelos_por_nodo.ipynb`) y la ablación sin homogeneidad (`02_SeleccionArquitectura.ipynb`). |
| **5. Evaluación** | ¿El modelo cumple el objetivo y generaliza? | Validación Out-Of-Fold $K=3$ ($r = 0.8584$, $R^2 = 0.7362$, $\text{MAE} = 296.3\text{ kg/ha}$) en `04_pipeline_agroraices.ipynb`. |
| **6. Despliegue** | ¿Cómo se explicarán y entregarán los resultados al usuario final? | Uso de **SHAP para IA Explicable (XAI)** en `04_pipeline_agroraices.ipynb` y creación como paso final del **Prototipo Web ([agro-raices-web.vercel.app](https://agro-raices-web.vercel.app/))** con el modelo y SHAP para dos tipos de usuario: **Agricultor** y **Analista**. |

---

## 1. Fase 1: Comprensión del Negocio y Doble Separación Neuro-Simbólica

El **Reto AgroCebada FIRA 2026** busca estimar el rendimiento de grano de cebada maltera (*Hordeum vulgare*, en $\text{t/ha}$) en 197 polígonos agrícolas de temporal (ciclo abril–octubre 2025) distribuidos en **Hidalgo**, **Puebla** y **Tlaxcala**.

Para resolver la **Heterogeneidad Espacial y Divergencia Biofísica con Covariables Homogéneas**, implementamos un **Marco de Argumentación Bipolar Cuantitativa (QBAF)** resuelto mediante el algoritmo **DF-QuAD** (*Rago, Cocarascu & Toni*) con **Doble Separación**:
1. **Separación en 5 Grupos Homogéneos de Variables (Nodos de Evidencia):** $G_{\text{Bio}}$, $G_{\text{Geo-Labranza}}$, $G_{\text{SWIR-Termo}}$, $G_{\text{Clima-PLS}}$ y $G_{\text{Meseta-Modal}}$.
2. **Separación por Estado y Grupos Homogéneos de Parcelas (Meta-Árbitros Regionales):**
   * **Hidalgo ($N=36$):** Contraste entre valles húmedos tecnificados y laderas semiáridas.
   * **Puebla ($N=71$):** Cuencas de alta densidad foliar con saturación óptica temprana ($\text{NDVI} > P_{75}$), donde una **Capa 0 de validación física (*Ataque Undercutting*)** anula el voto del árbol óptico ($\sigma_{\text{ET}} \to 0.001$).
   * **Tlaxcala ($N=31$):** Meseta con rendimientos reportados en escalones discretos múltiplos de $0.5\text{ t/ha}$ ($2.5, 3.0, 3.5\text{ t/ha}$).

---

## 2. Fase 2 y Fase 3: Comprensión y Preparación de los Datos 

Todas las fuentes ubicadas en `data/` se integran bajo prevención estricta de fuga de información (*Zero Data Leakage*):
* **Control Radiométrico (Capa 0):** Retención exclusiva de escenas con $\text{nubosidad} \le 30\%$ ($64.35\%$ en Sentinel-2/Landsat y $68.39\%$ en PlanetScope).
* **Geodesia y Morfometría (`data/shp/`):** Proyección a UTM Zona 14N (`EPSG:32614`), centroides (`lat_centro`, `lon_centro`), ratio de elongación (`elongacion_bbox`), altitud/pendiente (`elevacion_msnm`, `pendiente_grados`) y relieve local (`proxy_relieve_m`).
* **Índices Biofísicos y de Labranza Pre-Siembra:** Hidratación foliar (`s2_ndwi_mean`, `s2_ndwi_min`), banda SWIR no saturable (`s2_dswi2_mean`), clorofila en borde rojo (`s2_mra_mean`), rastrojo en pre-siembra enero–abril (`s2_sti_presow`, `s2_ndti_presow`), emisión térmica (`landsat_vi6t_max`, `landsat_vi6t_std`) y dinámica foliar a $3\text{ m}$ (`planet_lai_std`, `planet_msavi_mean`, `planet_ndvi_mean`, `persistencia_verde_m11`).
* **Downscaling Topográfico Adiabático ($\Gamma = -6.5^\circ\text{C}/\text{km}$):** Corrección física por altitud sobre el cubo climático `data/temperatura_chirts_era5_parcelas.csv`.

---

## 3. Fase 4: Modelado y Justificación Experimental (`notebooks/`)

### 3.1 La Naturaleza Matemática de los Datos como Punto de Partida (`notebooks/03_seleccion_modelos_por_nodo.ipynb`)

La búsqueda de modelos en cada nodo homogéneo parte del diagnóstico estadístico intrínseco de su matriz $\mathbf{X}_g$ y vector objetivo $\mathbf{y}_g$, validando después a los candidatos bajo validación cruzada ($K=3$), ruido sintético ($20\%$) e invariancia ante simulación de $20\%$ de nubes:

| Nodo QBAF (Grupo Homogéneo) | Naturaleza Matemática del Dato (Punto de Partida) | Modelo Seleccionado (Ganador) | Métricas del Ganador ($K=3$ OOF) | Métricas de Modelos Rivales Evaluados en el Nodo |
| :--- | :--- | :--- | :--- | :--- |
| **Nodo 1 ($G_{\text{Bio}}$ — Hidalgo)** | **Particiones por Umbral No Lineal:** Respuesta escalonada al frío (`bi_Tmin_anom25_04`) y borde rojo (`s2_mra_mean`). | **`GradientBoostingRegressor`**<br>*(+ Prior Bayes $m=0.5$)* | • **$R^2 = 0.8896$**<br>• **$\text{MAE} = 192.20\text{ kg/ha}$**<br>• Nubes $= 243.99\text{ kg/ha}$ | • `RandomForest`: $R^2=0.8717$, $\text{MAE}=216.43\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=0.8211$, $\text{MAE}=243.68\text{ kg/ha}$<br>• `Ridge`: $R^2=0.7859$, $\text{MAE}=288.90\text{ kg/ha}$<br>• `Huber`: $R^2=0.6734$, $\text{MAE}=330.41\text{ kg/ha}$<br>• `SVR RBF`: $R^2=0.6029$, $\text{MAE}=316.20\text{ kg/ha}$ |
| **Nodo 2 ($G_{\text{Geo-Labranza}}$ — Puebla)** | **Continuidad Espacial Suave:** Gradiente continuo de relieve (`proxy_relieve_m`) y labranza (`s2_sti_presow`). | **`SVR (Kernel RBF)` + `GPR (RBF + WhiteKernel)`** | • **`SVR`**: **$R^2 = 0.0910$**, **$\text{MAE} = 385.70\text{ kg/ha}$**<br>• **`GPR`**: $\text{MAE} = 399.83\text{ kg/ha}$ | • `Ridge`: $R^2=0.0364$, $\text{MAE}=410.99\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=0.0043$, $\text{MAE}=415.95\text{ kg/ha}$<br>• `RandomForest`: $R^2=-0.1994$, $\text{MAE}=473.40\text{ kg/ha}$<br>• `GradientBoosting`: $R^2=-0.2442$, $\text{MAE}=501.90\text{ kg/ha}$ |
| **Nodo 3 ($G_{\text{SWIR-Termo}}$ — Puebla)** | **Colas Pesadas y Outliers Térmicos:** Anomalías radiométricas en `landsat_vi6t_std` por bordes de nube. | **`HuberRegressor` ($\epsilon = 1.2$)** | • **$R^2 = 0.0664$**<br>• **$\text{MAE} = 365.56\text{ kg/ha}$**<br>• Nubes $= 361.46\text{ kg/ha}$ | • `Ridge`: $R^2=0.0571$, $\text{MAE}=397.97\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=0.0272$, $\text{MAE}=403.53\text{ kg/ha}$<br>• `SVR RBF`: $R^2=-0.0068$, $\text{MAE}=402.07\text{ kg/ha}$<br>• `RandomForest`: $R^2=-0.1103$, $\text{MAE}=444.59\text{ kg/ha}$ |
| **Nodo 4 ($G_{\text{Clima-PLS}}$ — Puebla)** | **Multicolinealidad Extrema ($\kappa(\mathbf{X}) \gg 30$):** Alta correlación climática mensual ($p \gg N$). | **`PLSRegression (k=2)` Supervisado** | • **$R^2 = 0.1103$**<br>• **$\text{MAE} = 403.70\text{ kg/ha}$** | • `PCA No Supervisado (k=2)`: $R^2=0.0378$, $\text{MAE}=430.69\text{ kg/ha}$<br>• `Sin Compresión (SVR)`: $R^2=0.0198$, $\text{MAE}=405.77\text{ kg/ha}$<br>• `Sin Compresión (ExtraTrees)`: $R^2=-0.1710$, $\text{MAE}=472.48\text{ kg/ha}$ |
| **Nodo 5 ($G_{\text{Meseta-Modal}}$ — Tlaxcala)** | **Target Discretizado ($Y$ en múltiplos de $0.5\text{ t/ha}$):** Rendimientos concentrados en escalones discretos. | **`SVC Ordinal Probabilístico` + Anclaje Modal ($3.0\text{ t/ha}$)** | • **$R^2 = 0.0552$**<br>• **$\text{MAE} = 274.77\text{ kg/ha}$**<br>• Nubes $= 298.32\text{ kg/ha}$ | • `SVR Continuo`: Colapsa con nubes a $380.46\text{ kg/ha}$<br>• `ExtraTrees`: $R^2=-0.0434$, $\text{MAE}=306.30\text{ kg/ha}$<br>• `RandomForest`: $R^2=-0.1068$, $\text{MAE}=313.05\text{ kg/ha}$<br>• `Huber`: $R^2=-0.4905$, $\text{MAE}=378.63\text{ kg/ha}$ |

---

### 3.2 Estudio de Ablación: ¿Por qué separar en Nodos Homogéneos y por qué se descartaron las Redes Neuronales? (`notebooks/02_SeleccionArquitectura.ipynb`)

En el cuaderno [`notebooks/02_SeleccionArquitectura.ipynb`](notebooks/02_SeleccionArquitectura.ipynb) se demuestra qué sucede cuando se entrenan modelos **separados por Estado pero mezclando las 39 variables sin dividir en nodos homogéneos**, incluyendo **3 arquitecturas de Redes Neuronales (`MLPRegressor`)**:

| Arquitectura / Modelo (Separado por Estado) | ¿Separa Nodos Homogéneos? | $R^2_{\text{Train}}$ Medio | $R^2_{\text{OOF}}$ Validación | Brecha *Overfitting* ($\Delta R^2$) | $\text{MAE}_{\text{Train}}$ ($\text{kg/ha}$) | $\text{MAE}_{\text{OOF}}$ Validación ($\text{kg/ha}$) | Pearson ($r$) OOF |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **QBAF + DF-QuAD (CON Nodos Homogéneos — GANADOR)** | **SÍ (5 Nodos + *Undercutting*)** | $0.5167$ | **$0.7362$** | **Sin sobreajuste** | $186.4$ | **$296.3$** | **$0.8584$** |
| **`ExtraTrees` (Sin Nodos Homogéneos)** | NO (39 vars mezcladas) | $0.9677$ | $0.6461$ | $+0.3216$ | $55.5$ | $375.7$ | $0.8067$ |
| **`RandomForest` (Sin Nodos Homogéneos)** | NO (39 vars mezcladas) | $0.8624$ | $0.6249$ | $+0.2375$ | $139.8$ | $379.9$ | $0.7933$ |
| **`GradientBoosting` (Sin Nodos Homogéneos)** | NO (39 vars mezcladas) | $0.9993$ | $0.5606$ | $+0.4387$ | $7.0$ | $396.7$ | $0.7592$ |
| **`SVR Kernel RBF` (Sin Nodos Homogéneos)** | NO (39 vars mezcladas) | $0.7105$ | $0.5702$ | $+0.1403$ | $195.9$ | $401.1$ | $0.7581$ |
| **`Ridge L2` (Sin Nodos Homogéneos)** | NO (39 vars mezcladas) | $0.6876$ | $0.5951$ | $+0.0925$ | $211.8$ | $410.8$ | $0.7722$ |
| **`HuberRegressor` (Sin Nodos Homogéneos)** | NO (39 vars mezcladas) | $0.8160$ | $0.3868$ | $+0.4292$ | $81.4$ | $488.9$ | $0.6725$ |
| **Red Neuronal Compacta `MLP (32-16, ReLU, L2=1.0)`** | NO (39 vars mezcladas) | **$0.9572$** | **$-0.0115$** | **$+0.9687$ (Overfitting)** | **$64.0$** | **$632.6$** | $0.5318$ |
| **Red Neuronal Profunda `MLP (128-64-32, ReLU)`** | NO (39 vars mezcladas) | **$0.9866$** | **$0.0415$** | **$+0.9452$ (Overfitting)** | **$26.7$** | **$637.0$** | $0.5246$ |
| **Red Neuronal Media `MLP (64-32, Tanh, L2=0.1)`** | NO (39 vars mezcladas) | **$0.9809$** | **$-0.1702$** | **$+1.1511$ (Overfitting)** | **$32.4$** | **$700.0$** | $0.5288$ |

---

## 4. Fase 5: Evaluación Oficial (`notebooks/04_pipeline_agroraices.ipynb`)

| Ámbito / Árbitro QBAF | Parcelas ($N$) | Correlación de Pearson ($r$) | Coeficiente de Determinación ($R^2$) | $\text{MAE}$ ($\text{t/ha}$) | $\text{MAE}$ ($\text{kg/ha}$) | $\text{RMSE}$ ($\text{kg/ha}$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Nacional (Global QBAF + DF-QuAD)** | **$138$** | **$0.8584$** | **$0.7362$** | **$0.2963$** | **$296.3\text{ kg/ha}$** | **$435.6\text{ kg/ha}$** |
| **Árbitro QBAF Hidalgo** | $36$ | **$0.9436$** | **$0.8885$** | **$0.1882$** | **$188.2\text{ kg/ha}$** | **$256.0\text{ kg/ha}$** |
| **Árbitro QBAF Puebla** | $71$ | **$0.4433$** | **$0.1710$** | **$0.3605$** | **$360.5\text{ kg/ha}$** | **$528.3\text{ kg/ha}$** |
| **Árbitro QBAF Tlaxcala** | $31$ | **$0.3821$** | **$0.0552$** | **$0.2748$** | **$274.8\text{ kg/ha}$** | **$359.6\text{ kg/ha}$** |

---

## 5. Fase 6: Despliegue — Uso de SHAP para XAI y Prototipo Web para Agricultor y Analista

La **Fase 6 (Despliegue)** dentro de nuestra metodología CRISP-DM se compone de dos etapas complementarias que conectan el cuaderno principal con la aplicación real en campo:

### 5.1 Etapa 1 en `notebooks/04_pipeline_agroraices.ipynb`: Uso de SHAP para Inteligencia Artificial Explicable (XAI)

En lugar de entregar un número de caja negra, en la Sección 6 de [`notebooks/04_pipeline_agroraices.ipynb`](notebooks/04_pipeline_agroraices.ipynb) se implementó **SHAP (`shap.Explainer`)** como motor de **Inteligencia Artificial Explicable (XAI)** acoplado a la deliberación **QBAF + DF-QuAD**. Cada predicción ciega (`CONJUNTO == 'PREDICCION'`) se descompone como una balanza aditiva exacta $\hat{y}(x) = \phi_0 + \sum_{j=1}^M \phi_j(x)$:

| Estado | Parcela (`ID_POLIGONO`) | Base $\phi_0$ ($\text{t/ha}$) | Top 1 Variable (Peso $\text{kg/ha}$ \| $\%$ Importancia) | Top 2 Variable (Peso $\text{kg/ha}$ \| $\%$ Importancia) | Top 3 Variable (Peso $\text{kg/ha}$ \| $\%$ Importancia) | Predicción Final $\hat{y}$ ($\text{t/ha}$) |
| :--- | :---: | :---: | :--- | :--- | :--- | :---: |
| **Hidalgo** | **`AGC_003`** *(18.68 ha)* | $3.7669$ | `bi_Tmin_anom25_04`: **$-211.2\text{ kg/ha}$ ($29.4\%$)** | `prior_municipal`: **$-135.9\text{ kg/ha}$ ($18.9\%$)** | `s2_mra_mean`: **$+117.1\text{ kg/ha}$ ($16.3\%$)** | **$3.3876\text{ t/ha}$** *($63.28\text{ t}$)* |
| **Hidalgo** | **`AGC_030`** *(13.05 ha)* | $3.7669$ | `bi_Tmin_anom25_04`: **$-226.7\text{ kg/ha}$ ($30.1\%$)** | `prior_municipal`: **$-151.7\text{ kg/ha}$ ($20.1\%$)** | `lp_Tmin_anom25_04`: **$-96.4\text{ kg/ha}$ ($12.8\%$)** | **$3.1207\text{ t/ha}$** *($40.73\text{ t}$)* |
| **Hidalgo** | **`AGC_042`** *(3.81 ha)* | $3.7669$ | `bi_Tmin_anom25_04`: **$-421.8\text{ kg/ha}$ ($37.7\%$)** | `lp_Tmin_anom25_04`: **$-292.7\text{ kg/ha}$ ($26.2\%$)** | `prior_municipal`: **$-166.3\text{ kg/ha}$ ($14.9\%$)** | **$2.7448\text{ t/ha}$** *($10.46\text{ t}$)* |
| **Puebla** | **`AGC_011`** *(7.17 ha)* | $4.5346$ | `log_area`: **$-52.6\text{ kg/ha}$ ($15.2\%$)** | `landsat_vi6t_std`: **$+48.6\text{ kg/ha}$ ($14.1\%$)** | `Z1_pls_clima`: **$+44.2\text{ kg/ha}$ ($12.8\%$)** | **$4.7076\text{ t/ha}$** *($33.75\text{ t}$)* |
| **Puebla** | **`AGC_016`** *(7.47 ha)* | $4.5346$ | `s2_sti_presow`: **$-67.5\text{ kg/ha}$ ($29.2\%$)** | `log_area`: **$-65.8\text{ kg/ha}$ ($28.4\%$)** | `elevacion_msnm`: **$-23.0\text{ kg/ha}$ ($9.9\%$)** | **$4.4164\text{ t/ha}$** *($32.99\text{ t}$)* |
| **Puebla** | **`AGC_028`** *(1.86 ha)* | $4.5346$ | `s2_ndwi_min`: **$+105.5\text{ kg/ha}$ ($23.4\%$)** | `log_area`: **$+59.0\text{ kg/ha}$ ($13.1\%$)** | `s2_sti_presow`: **$+58.5\text{ kg/ha}$ ($13.0\%$)** | **$4.6546\text{ t/ha}$** *($8.66\text{ t}$)* |
| **Tlaxcala** | **`AGC_025`** *(2.00 ha)* | $2.9941$ | `elongacion_bbox`: **$+6.4\text{ kg/ha}$ ($39.4\%$)** | `planet_msavi_mean`: **$+3.9\text{ kg/ha}$ ($24.2\%$)** | `landsat_intra_vi6t_std`: **$+3.7\text{ kg/ha}$ ($23.0\%$)** | **$3.0065\text{ t/ha}$** *($6.01\text{ t}$)* |
| **Tlaxcala** | **`AGC_027`** *(8.46 ha)* | $2.9941$ | `planet_msavi_mean`: **$-12.2\text{ kg/ha}$ ($55.6\%$)** | `landsat_intra_vi6t_std`: **$-4.8\text{ kg/ha}$ ($21.8\%$)** | `lat_centro`: **$+2.7\text{ kg/ha}$ ($12.3\%$)** | **$2.9805\text{ t/ha}$** *($25.21\text{ t}$)* |
| **Tlaxcala** | **`AGC_029`** *(5.90 ha)* | $2.9941$ | `planet_msavi_mean`: **$+7.9\text{ kg/ha}$ ($29.8\%$)** | `lon_centro`: **$+7.8\text{ kg/ha}$ ($29.4\%$)** | `elongacion_bbox`: **$+5.6\text{ kg/ha}$ ($21.2\%$)** | **$3.0206\text{ t/ha}$** *($17.82\text{ t}$)* |

---

### 5.2 Paso Final del Despliegue: Creación del Prototipo Web con el Modelo y SHAP para Dos Tipos de Usuario ([`https://agro-raices-web.vercel.app/`](https://agro-raices-web.vercel.app/))

Como paso final del despliegue, se creó el prototipo interactivo **[AgroRaíces Web (`https://agro-raices-web.vercel.app/`)](https://agro-raices-web.vercel.app/)**, el cual integra el modelo **QBAF + DF-QuAD** y las explicaciones **SHAP (XAI)** adaptadas específicamente para **dos tipos de usuario**:

| Tipo de Usuario | Cómo presenta el Prototipo el Modelo + SHAP (XAI) | Beneficio Directo |
| :--- | :--- | :--- |
| ** 1. Agricultor (Productor de Campo)** | • **Estimación clara de cosecha:** Muestra el rendimiento estimado ($\text{t/ha}$) y las toneladas/costales esperados en su parcela.<br>• **Traducción de SHAP a lenguaje de campo:** Convierte los valores SHAP en explicaciones sencillas y sin tecnicismos (*qué ayudó a tu cultivo* —como humedad en hojas o cobertura de rastrojo— y *qué factores jugaron en contra* —como mañanas frías en abril o pendiente del terreno—).<br>• **Asesoría práctica:** Recomendaciones agronómicas puntuales para cuidar el ciclo. | Permite al productor entender el motivo del cálculo y anticipar decisiones de manejo y logística de trilla. |
| **2. Analista (Técnico FIRA / Investigador)** | • **Desglose técnico de la Balanza SHAP:** Presenta los pesos exactos $\phi_j(x)$ en $\text{kg/ha}$, el valor base regional $\phi_0$ y el porcentaje de impacto de cada variable biofísica, climática y topográfica.<br>• **Deliberación del Consejo de Expertos (QBAF + DF-QuAD):** Muestra la contribución de los nodos especialistas por Estado y la activación de reglas físicas (*Undercutting*).<br>• **Mapa geoespacial interactivo:** Consulta por polígono y análisis regional en Hidalgo, Puebla y Tlaxcala. | Garantiza transparencia matemática total para validación técnica, planeación financiera y seguimiento regional. |



