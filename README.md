# Clasificación de eventos H→ττ con el ATLAS Higgs Challenge 2014

Este proyecto busca distinguir los eventos de señal (desintegración del bosón de Higgs en dos leptones tau, H→ττ) de los eventos de fondo, usando los datos del **ATLAS Higgs Challenge 2014**. 

Primero se estudia con un mapa autoorganizado (SOM) si las dos clases se separan de forma natural. Después se entrenan modelos supervisados (regresión logística y perceptrón multicapa, MLP) y, por último, se usa un algoritmo genético para optimizar la métrica de significancia **AMS** (*Approximate Median Significance*).

- **Autora:** _Claudia García Navas_
- **Asignatura:** Aprendizaje Automático
- **Universidad:** Universidad Francisco de Vitoria (UFV)
- **Curso:** 3.º de carrera, curso académico _[2026/2027]_

Este proyecto busca distinguir los eventos de señal (desintegración del bosón de Higgs en dos leptones tau, H→ττ) de los eventos de fondo, usando los datos del **ATLAS Higgs Challenge 2014**. 

Primero se estudia con un mapa autoorganizado (SOM) si las dos clases se separan de forma natural. Después se entrenan modelos supervisados (regresión logística y perceptrón multicapa, MLP) y, por último, se usa un algoritmo genético para optimizar la métrica de significancia **AMS** (*Approximate Median Significance*).

---


## 1. Dataset

> **El dataset no está incluido en el repositorio.** El CSV original (~195 MB) y los CSV intermedios que se generan a partir de él están excluidos en `.gitignore`. Hay que descargarlo antes de ejecutar nada.

### Fuente

- **ATLAS Higgs Challenge 2014**, publicado por la colaboración ATLAS en el CERN Open Data Portal.
- Registro: <http://opendata.cern.ch/record/328>
- DOI: [10.7483/OPENDATA.ATLAS.ZBP2.M5T8](https://doi.org/10.7483/OPENDATA.ATLAS.ZBP2.M5T8)

### Descarga

1. Entra en el registro del CERN Open Data Portal (enlace de arriba).
2. Descarga el archivo `atlas-higgs-challenge-2014-v2.csv.gz` y descomprímelo.
3. Coloca el CSV en la **raíz del repositorio** con este nombre exacto:

   ```
   atlas-higgs-challenge-2014-v2.csv
   ```

### Licencia

Los datos se distribuyen bajo licencia **Creative Commons CC0** (dominio público), como el resto de los datos del CERN Open Data Portal. Aun así, se debe citar la fuente (ver [Referencias](#9-referencias)).

### Notas clave sobre los datos

- **Tamaño:** 818 238 eventos y 35 columnas: un identificador (`EventId`), 30 variables físicas (17 primitivas `PRI_*` y 13 derivadas `DER_*`), y las columnas `Weight`, `Label`, `KaggleSet` y `KaggleWeight`.
- **`Label`:** `s` = señal (Higgs) y `b` = fondo.
- **`KaggleSet`:** indica a qué subconjunto de la competición original de Kaggle pertenece cada evento. En este proyecto se usa así para hacer la partición:

  | Valor | Uso en la competición | Uso en este proyecto | Eventos |
  |---|---|---|---|
  | `t` | Entrenamiento | **Entrenamiento** | 250 000 |
  | `b` | Leaderboard público | **Ajuste** (hiperparámetros) | 100 000 |
  | `v` | Leaderboard privado | **Test** | 450 000 |
  | `u` | No usado | Se descarta | — |

- **`KaggleWeight`:** es el peso de cada evento, normalizado dentro de su subconjunto de `KaggleSet`. Es el peso que se usa para calcular la AMS en cada partición. `Weight` es el mismo peso pero normalizado sobre el dataset completo.
- **Valores −999:** significan **"variable no definida"**, no datos perdidos. Por ejemplo, las variables del segundo chorro (`PRI_jet_subleading_*`) valen −999 cuando el evento tiene menos de dos chorros (`PRI_jet_num` < 2). Estos eventos **no se eliminan**, porque el número de chorros está relacionado con el modo de producción del Higgs (fusión de gluones frente a fusión de bosones vectoriales, VBF).



## 2. Estructura del repositorio

| Archivo | Descripción |
|---|---|
| [1_EDA.ipynb](1_EDA.ipynb) | Análisis exploratorio: estudio de los −999, distribuciones, correlaciones, asimetría, transformación logarítmica, estandarización y partición en Entrenamiento / Ajuste / Test. Genera los CSV intermedios. |
| [2_SOM.ipynb](2_SOM.ipynb) | Mapa de Kohonen (SOM) programado desde cero con NumPy (no supervisado). Estudia si señal y fondo se agrupan de forma natural, con las variables PRI y con PRI + DER. |
| [3_MLP_RegresiónLogistica.ipynb](3_MLP_RegresiónLogistica.ipynb) | Modelos supervisados: regresión logística (modelo base), definición de la AMS y MLP. |
| [4_AlgoritmoGenetico.ipynb](4_AlgoritmoGenetico.ipynb) | Algoritmo genético para optimizar la AMS. |
| [requirements.txt](requirements.txt) | Dependencias de Python. |
| [.gitignore](.gitignore) | Excluye el CSV original, los CSV intermedios, entornos virtuales y archivos temporales. |
| [.gitattributes](.gitattributes) | Normaliza los finales de línea de los archivos de texto. |

**Archivos generados (no versionados):** `Entrenamiento.csv`, `Ajuste.csv` y `Test.csv`. Los crea `1_EDA.ipynb` en la raíz del repositorio.



## 3. Instalación

Se ha usado **Python 3.13** (kernel de Jupyter con Python 3.13.5).

1. Clona el repositorio y entra en la carpeta.
2. (Opcional, pero recomendable) Crea y activa un entorno virtual:

   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # Linux / macOS
   source .venv/bin/activate
   ```

3. Instala las dependencias:

   ```bash
   pip install -r requirements.txt
   ```

Librerías principales y versiones que se han usado: `numpy` 2.3, `pandas` 2.3, `scikit-learn` 1.9, `matplotlib` 3.10 y `jupyter`.



## 4. Cómo reproducir los resultados

### Orden de ejecución

Los cuadernos se ejecutan **en orden, de 1 a 4**:

1. **`1_EDA.ipynb`**: lee `atlas-higgs-challenge-2014-v2.csv` y genera `Entrenamiento.csv`, `Ajuste.csv` y `Test.csv` (ya transformados y estandarizados). **Hay que ejecutarlo primero**: los demás cuadernos dependen de estos CSV.
2. **`2_SOM.ipynb`**: lee `Entrenamiento.csv`.
3. **`3_MLP_RegresiónLogistica.ipynb`**: lee `Entrenamiento.csv` (y `Ajuste.csv` / `Test.csv` para el ajuste y la evaluación). _En desarrollo._
4. **`4_AlgoritmoGenetico.ipynb`**: _En desarrollo._

### Semillas aleatorias

- SOM: `np.random.default_rng(42)`. La misma semilla sirve para inicializar los pesos y para barajar los eventos en cada época.
- MLP, regresión logística y algoritmo genético: _Pendiente, pero también se usará la semilla 42_.

### Tiempo de ejecución y hardware

- `1_EDA.ipynb`: pocos minutos. Lo más lento es cargar el CSV y dibujar las figuras.
- `2_SOM.ipynb`: **es el más pesado**. El entrenamiento es un bucle en Python puro (20 épocas × 250 000 eventos, para dos mapas: PRI y PRI + DER). _[Tiempo aproximado: pendiente de medir y de las épocas]_.
- `4_AlgoritmoGenetico.ipynb`: _[pendiente]_.
- **No hace falta GPU.** Todo el código usa NumPy y scikit-learn en CPU.



## 5. Metodología resumida

**Preprocesado (`1_EDA.ipynb`)**

- Los −999 se convierten en `NaN` para que no deformen las distribuciones ni las correlaciones.
- Partición según `KaggleSet` (`t` / `b` / `v`) en Entrenamiento, Ajuste y Test.
- Transformación logarítmica de las variables con cola larga (las que tienen mucha asimetría): `log(x)` si la variable no tiene ceros y `log(1+x)` si los tiene. No se aplica a las variables simétricas, a las acotadas (*centrality*) ni a los pesos.
- Estandarización con `StandardScaler`, ajustado **solo con el conjunto de Entrenamiento** y aplicado después a Ajuste y Test.
- En los cuadernos de modelos, los `NaN` (los antiguos −999) se sustituyen por 0. Tras estandarizar, 0 corresponde a la media de la variable.
- Se trabaja con dos conjuntos de variables: **solo PRI** (17 variables) y **PRI + DER** (30 variables).

**SOM (`2_SOM.ipynb`)**

- Rejilla de 20×20 neuronas, vecindad gaussiana, radio que baja de 10 a 1 y tasa de aprendizaje que baja de 0,1 a 0,01 (las dos con decaimiento exponencial).
- Visualizaciones: hit maps por clase, U-matrix con fronteras (percentil 75), proporción de señal por neurona y planos de componentes.

**Modelos supervisados (`3_MLP_RegresiónLogistica.ipynb`)**

- Regresión logística como modelo base y MLP, entrenados con PRI y con PRI + DER.
- Las etiquetas se codifican como `s` → 1 y `b` → 0.
_En desarrollo_

**Métrica AMS**

La AMS es la métrica oficial del challenge:

$$\text{AMS} = \sqrt{2\left((s + b + b_r)\ln\left(1 + \frac{s}{b + b_r}\right) - s\right)}, \qquad b_r = 10$$

donde $s$ y $b$ son la suma de los pesos (`KaggleWeight`) de los eventos de señal y de fondo que el modelo clasifica como señal.

**Algoritmo genético (`4_AlgoritmoGenetico.ipynb`)**

- Optimización de la AMS.
_En desarrollo_



## 6. Resultados

- **EDA:** tras aplicar los logaritmos y la estandarización, las distribuciones de señal y de fondo se diferencian mejor en varias variables. Los −999 tienen una estructura clara: el 40 % de los eventos no tiene ningún chorro y el 70,9 % no tiene un segundo chorro.
- **SOM:** **señal y fondo no se separan de forma natural.** En los hit maps, las dos clases se reparten de forma casi homogénea por el mapa, y las zonas con más señal no coinciden con las fronteras de la U-matrix. Esto pasa tanto con PRI como con PRI + DER. Por tanto, hacen falta modelos supervisados.
- **Regresión logística, MLP y algoritmo genético:** en desarrollo.

**Entrega final** (se completará):

| Modelo | Variables | AMS (Test) | Accuracy | AUC-ROC | Otras |
|---|---|---|---|---|---|
| Regresión logística | PRI | — | — | — | — |
| Regresión logística | PRI + DER | — | — | — | — |
| MLP | PRI | — | — | — | — |
| MLP | PRI + DER | — | — | — | — |
| MLP + algoritmo genético | PRI + DER | — | — | — | — |



## 7. Estado del proyecto

**Hecho**

- [x] Análisis exploratorio completo y estudio de los valores −999.
- [x] Transformación logarítmica, estandarización y partición en Entrenamiento / Ajuste / Test.
- [x] SOM programado desde cero, entrenado y analizado (PRI y PRI + DER).

**Qué falta por hacer**

- [ ] Regresión logística (modelo base).
- [ ] Implementar la función AMS.
- [ ] MLP: arquitectura, entrenamiento y ajuste de hiperparámetros con el conjunto de Ajuste.
- [ ] Algoritmo genético para optimizar la AMS.
- [ ] Evaluación final en Test y tabla comparativa de modelos.
- [ ] Completar `requirements.txt`, fijar todas las semillas y medir los tiempos de ejecución.
- [] Completar `README`

---

## 8. Uso de IA

De acuerdo con la sección de uso ético de la IA de la asignatura, declaro que he usado herramientas de inteligencia artificial generativa como apoyo durante el proyecto: para resolver dudas conceptuales, revisar código erróneo y preparar visualizaciones. Todas las decisiones de diseño, el análisis de los resultados y las conclusiones son míos, y he revisado y entendido todo el código del repositorio. _[Modelo: Claude Opus 5.5 Medio]_.



## 9. Referencias

- **Dataset:** ATLAS Collaboration (2014). *Dataset from the ATLAS Higgs Boson Machine Learning Challenge 2014*. CERN Open Data Portal. DOI: [10.7483/OPENDATA.ATLAS.ZBP2.M5T8](https://doi.org/10.7483/OPENDATA.ATLAS.ZBP2.M5T8)
- **Paper del challenge (definición de la AMS):** Adam-Bourdarios, C., Cowan, G., Germain, C., Guyon, I., Kégl, B. y Rousseau, D. (2014). *Learning to discover: the Higgs boson machine learning challenge*. Documentación técnica del HiggsML Challenge. Disponible en <http://opendata.cern.ch/record/328>.
- Documentación de las librerías:
  - NumPy: <https://numpy.org/doc/>
  - pandas: <https://pandas.pydata.org/docs/>
  - scikit-learn: <https://scikit-learn.org/stable/>
  - Matplotlib: <https://matplotlib.org/stable/>
