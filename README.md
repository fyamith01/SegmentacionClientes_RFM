# Segmentación de Clientes mediante Análisis RFM y Clustering

![Portada del Proyecto](inputs/images/portada.png)

> **Autor:** Fabián Yamith Tovar Altahona — fabianyamith123@gmail.com  
> **Licencia:** MIT

---

## Descripción General

Este proyecto implementa un sistema de **segmentación de clientes** utilizando el modelo **RFM (Recency, Frequency, Monetary)** combinado con algoritmos de **clustering (K-Means y DBSCAN)** para identificar grupos de clientes y asignarles descuentos diferenciados del **5%, 20% y 25%** según su comportamiento transaccional.

El análisis se realiza sobre una base de datos relacional que contiene las transacciones de los últimos seis meses de un grupo de clientes, incluyendo información sobre la clase de transacción (`COMPRA` / `AVANCE`), el valor monetario y la fecha.

---

## Estructura

```
SegmentacionClientes_RFM/
│
├── 📓 DataAnalisis.ipynb          # Notebook principal con todo el análisis
├── 📄 LICENSE                      # Licencia MIT del proyecto
├── 📄 requierements.txt           # Dependencias de Python del proyecto
├── 📄 README.md                    # Este documento
│
├── 📁 inputs/                      # Datos de entrada (fuente original)
│   ├── BaseDeDatos_y_PreguntasSQL.xlsx   # Base de datos con dos hojas/tablas
│   └── 📁 images/                  # Imágenes de apoyo
│       ├── portada.png             # Imagen de portada del notebook
│       └── query1_sql.jpg ~ query7_sql.jpg  # Capturas de consultas SQL
│
├── 📁 intermediate/                # Datos intermedios (generados por el notebook)
│   ├── data_merged_cleaned.xlsx    # DataFrame unido y limpio (sin nulos)
│   └── rfm_clientes_cleaned.xlsx   # Métricas RFM por cliente (agrupadas)
│
└── 📁 outputs/                     # Resultados finales
    └── rfm_clientes_clusters.xlsx  # Clientes con segmentos y descuentos asignados
```

---

## Descripción de Directorios

### `inputs/` — Datos de Entrada

Contiene los datos originales que alimentan el análisis:

| Archivo | Descripción |
|---------|-------------|
| `BaseDeDatos_y_PreguntasSQL.xlsx` | Libro de Excel con **dos hojas**: `Detalle_cliente` (Id_cliente, fecha_efectiva, Id_tx) y `Detalle_tx` (Id_tx, clase, valor). Contiene **2500 registros** por tabla. |
| `images/portada.png` | Imagen decorativa usada como portada del notebook. |
| `images/query1_sql.jpg` ~ `query7_sql.jpg` | Capturas de las consultas SQL ejecutadas en un gestor de bases de datos externo, usadas como evidencia visual en la sección 2 del notebook. |

### `intermediate/` — Datos Intermedios

Archivos generados durante el procesamiento del notebook:

| Archivo | Descripción |
|---------|-------------|
| `data_merged_cleaned.xlsx` | Resultado de unir (`merge`) las tablas de clientes y transacciones, tras limpieza de fechas, tipado de datos y eliminación de nulos. Contiene **3277 registros** válidos. |
| `rfm_clientes_cleaned.xlsx` | Datos agrupados **por cliente** con las métricas RFM calculadas: Recency (días desde última compra), Frequency (total de transacciones) y Monetary (valor total gastado). |

### `outputs/` — Resultados Finales

| Archivo | Descripción |
|---------|-------------|
| `rfm_clientes_clusters.xlsx` | Resultado final del análisis: cada cliente con sus métricas RFM, el **cluster/segmento** asignado por K-Means y el **porcentaje de descuento** correspondiente (5%, 20% o 25%). |

### Archivos Raíz

| Archivo | Descripción |
|---------|-------------|
| `DataAnalisis.ipynb` | **Notebook principal** — Contiene todo el código, análisis y visualizaciones. Se detalla a continuación. |
| `LICENSE` | Licencia MIT (código abierto). |
| `requierements.txt` | Lista de dependencias de Python necesarias para ejecutar el notebook. |

---

## Descripción Detallada del Notebook `DataAnalisis.ipynb`

El notebook está organizado en **dos grandes secciones** con múltiples sub-secciones:

---

### Sección 1: Habilidad Práctica — Análisis de Datos

Esta sección comprende el **análisis exploratorio (EDA)** y la **segmentación de clientes con RFM + Clustering**.

#### 1.1 Análisis Exploratorio de los Datos (EDA)

| Sub-sección | Descripción | Qué modificar si... |
|---|---|---|
| **1.1.1 Cargar Datos** | Lee las dos hojas del archivo Excel (`Detalle_cliente` y `Detalle_tx`) en dos DataFrames. Limpia espacios en blanco y convierte `Id_tx` a numérico para garantizar la unión correcta. | Cambiar la **fuente de datos**: modificar la ruta del archivo en `pd.read_excel()` y los nombres de las hojas (`sheet_name`). Si los datos provienen de otra fuente (CSV, base de datos), cambiar la función de lectura. |
| **1.1.2 Formato de fecha adecuado** | Procesa la columna `fecha_efectiva` (formato `YYYYMMDD` como string). Fechas con 6 caracteres (`YYYYMM`) se convierten al día 1 del mes. Fechas con menos de 6 o nulas se descartan. | Cambiar el **formato de fecha**: ajustar la función `format_fecha()` y el formato en `pd.to_datetime(format='...')`. |
| **1.1.3 Unión de los dos DataFrames** | Realiza un `LEFT JOIN` sobre `Id_tx` para mantener todos los registros de clientes, incluso los que no tienen transacciones. | Cambiar la **clave de unión** o el **tipo de join**: modificar los parámetros `on=` y `how=` en `df_cliente.merge()`. |
| **1.1.4 Terminamos de tipar los datos** | Convierte `fecha_efectiva` a `datetime`, `valor` a `float64`, `clase` a `category`. | Cambiar los **tipos de datos**: ajustar las conversiones con `pd.to_datetime()`, `pd.to_numeric()`, `.astype()`. |
| **1.1.5 Breve descripción de las variables** | Ejecuta `df_merged.describe()` para obtener estadísticas descriptivas. Identifica la dispersión en la variable `valor` y un dato fuera de rango en las fechas. | No requiere modificación directa; es un paso de inspección. |
| **1.1.6 Filtrar 6 meses atrás** | Filtra los datos para conservar solo los registros dentro de los últimos 6 meses respecto a la fecha máxima (`2021-03-08`). Se usa `pd.DateOffset(months=6)`. | Cambiar el **período de análisis**: ajustar el valor de `months=6` en `pd.DateOffset()` o cambiar la fecha de referencia. |
| **1.1.7 Verificar datos no consistentes y nulos** | Verifica que no haya valores negativos en `valor`. Elimina registros con `valor` o `clase` nulos. Resultado: **3277 registros limpios**. | Cambiar las **reglas de limpieza**: ajustar los filtros booleanos que definen qué registros se conservan. |
| **1.1.8 Visualizaciones antes de agrupar** | Genera gráficos como boxplots de `valor` por `clase` de transacción usando `matplotlib` y `seaborn`. | Cambiar los **gráficos**: modificar las llamadas a `sns.boxplot()`, `plt.figure()`, etc. |
| **1.1.9 Feature Selection** | Agrupa los datos por `Id_cliente` para calcular las métricas RFM: **Recency** (días desde la última transacción), **Frequency** (conteo de transacciones) y **Monetary** (suma total del valor). Guarda en `intermediate/rfm_clientes_cleaned.xlsx`. | Cambiar las **variables de segmentación**: modificar las funciones de agregación en `groupby()` (ej. usar mediana en lugar de suma para Monetary, o agregar nuevas métricas). |
| **1.1.10 Verificar nulos y outliers** | Revisa nulos y outliers en el nuevo DataFrame agrupado por cliente. | Ajustar el tratamiento de outliers si se desea un enfoque diferente (ej. IQR, z-score, etc.). |
| **1.1.11 Gráficos de análisis por cliente** | Visualizaciones interactivas con `plotly` (histogramas, boxplots, scatter plots) para analizar las distribuciones de R, F y M por cliente. | Cambiar los **tipos de gráficos** o librerías de visualización. |

#### 1.2 Segmentación de Clientes RFM

| Sub-sección | Descripción | Qué modificar si... |
|---|---|---|
| **1.2.1 Estandarización de datos RFM** | Aplica `RobustScaler` de `sklearn` a las variables RFM para escalarlas antes del clustering. Se usa `RobustScaler` en lugar de `StandardScaler` porque es más robusto ante outliers. También aplica transformación `log1p` a Monetary por su alta dispersión. | Cambiar el **método de escalado**: reemplazar `RobustScaler` por `StandardScaler`, `MinMaxScaler`, etc. Cambiar la **transformación** de variables (ej. quitar `log1p` o aplicar otra). |
| **1.2.2 Aplicando K-Means** | Determina el número óptimo de clusters (K) usando el **método del codo (Elbow Method)**, **Silhouette Score**, **Calinski-Harabasz Score** y **Davies-Bouldin Score**. Ejecuta K-Means con el K seleccionado y visualiza los clusters con PCA (reducción a 2D). | Cambiar el **número de clusters**: ajustar `n_clusters` en `KMeans()`. Cambiar el **rango de búsqueda**: modificar el rango del bucle que evalúa distintos K. Cambiar las **métricas de evaluación**. |
| **1.2.3 Segmentos y Descuentos** | Asigna etiquetas descriptivas y descuentos a cada cluster en función del comportamiento RFM. Se definen **3 segmentos**: clientes de alta, media y baja actividad con descuentos del 5%, 20% y 25% respectivamente. Guarda el resultado en `outputs/rfm_clientes_clusters.xlsx`. | Cambiar los **porcentajes de descuento**: modificar el mapeo cluster → descuento. Cambiar las **etiquetas de los segmentos**. Cambiar la **lógica de asignación** (ej. usar reglas de negocio diferentes). |
| **1.2.4 Un vistazo a DBSCAN** | Aplica el algoritmo DBSCAN como alternativa a K-Means. Compara resultados y explica por qué K-Means fue más adecuado para este caso de uso particular. | Experimentar con los **hiperparámetros de DBSCAN** (`eps`, `min_samples`) para obtener diferentes agrupaciones. |

---

### Sección 2: Ejecución de Consultas SQL

Esta sección contiene **7 consultas SQL** realizadas sobre una base de datos relacional distinta (presumiblemente de estudiantes universitarios). Cada consulta se resuelve en SQL y se acompaña de una captura de pantalla con la evidencia de ejecución.

| Consulta | Descripción |
|----------|-------------|
| **2.1** | Cantidad de estudiantes por ciudad. |
| **2.2** | Agrupación de estudiantes por año de ingreso y carrera. |
| **2.3** | Meses transcurridos desde una fecha de referencia. |
| **2.4** | Ciudades sin estudiantes registrados. |
| **2.5** | Estudiantes que no tienen teléfono registrado. |
| **2.6** | Cantidad de estudiantes por carrera en cada ciudad. |
| **2.7** | Validación de consistencia del campo `CORREO`. |

> **Nota:** Las capturas de las queries se encuentran en `inputs/images/query1_sql.jpg` a `query7_sql.jpg`.

---

## Tecnologías y Librerías Utilizadas

| Categoría | Librerías |
|-----------|-----------|
| **Manipulación de datos** | `pandas`, `numpy` |
| **Visualización estática** | `matplotlib`, `seaborn` |
| **Visualización interactiva** | `plotly` (`express`, `graph_objects`, `figure_factory`, `subplots`) |
| **Machine Learning / Clustering** | `scikit-learn` (`KMeans`, `DBSCAN`, `PCA`, `StandardScaler`, `RobustScaler`, `silhouette_score`, `calinski_harabasz_score`, `davies_bouldin_score`) |
| **Estadística** | `scipy` (`gaussian_kde`, `cdist`) |
| **Utilidades** | `datetime`, `warnings` |

---

## Cómo Ejecutar

### Prerrequisitos

1. **Python 3.8+** instalado.
2. **Jupyter Notebook** o **JupyterLab**.

### Instalación

```bash
# Clonar el repositorio
git clone https://github.com/fyamith01/SegmentacionClientes_RFM.git
cd SegmentacionClientes_RFM

# Instalar dependencias
pip install -r requierements.txt

# Ejecutar el notebook
jupyter notebook DataAnalisis.ipynb
```

### Orden de ejecución

Ejecutar las celdas del notebook **en orden secuencial** (de arriba hacia abajo). El notebook genera automáticamente:

1. `intermediate/data_merged_cleaned.xlsx` — datos limpios y unidos.
2. `intermediate/rfm_clientes_cleaned.xlsx` — métricas RFM por cliente.
3. `outputs/rfm_clientes_clusters.xlsx` — segmentación final con descuentos.

---

## Flujo del Pipeline de Datos

```mermaid
flowchart LR
    A["📥 inputs/\nBaseDeDatos_y_\nPreguntasSQL.xlsx"] --> B["📊 DataAnalisis.ipynb"]
    B --> C["📂 intermediate/\ndata_merged_cleaned.xlsx"]
    C --> B
    B --> D["📂 intermediate/\nrfm_clientes_cleaned.xlsx"]
    D --> B
    B --> E["📤 outputs/\nrfm_clientes_clusters.xlsx"]
```

**Pasos del pipeline:**

1. **Ingesta**: Lectura de las dos tablas del Excel de entrada.
2. **Limpieza**: Formateo de fechas, tipado, eliminación de nulos e inconsistencias.
3. **Unión**: Merge LEFT JOIN por `Id_tx`.
4. **Filtrado temporal**: Solo últimos 6 meses.
5. **EDA**: Visualizaciones y estadísticas descriptivas.
6. **Agregación RFM**: Cálculo de Recency, Frequency y Monetary por cliente.
7. **Escalado**: `RobustScaler` + transformación `log1p`.
8. **Clustering**: K-Means (con evaluación del K óptimo) y DBSCAN.
9. **Asignación de descuentos**: Mapeo cluster → descuento (5%, 20%, 25%).
10. **Exportación**: Resultado final en `outputs/rfm_clientes_clusters.xlsx`.

---

## Guía Rápida:

| Objetivo | Dónde modificar |
|----------|-----------------|
| Usar una **fuente de datos diferente** | Sección 1.1.1 — Cambiar `pd.read_excel()` por la función de lectura correspondiente. |
| Cambiar el **período de análisis** | Sección 1.1.6 — Ajustar `pd.DateOffset(months=6)`. |
| Agregar o cambiar **variables al modelo RFM** | Sección 1.1.9 — Modificar las agregaciones en `groupby()`. |
| Cambiar el **método de escalado** | Sección 1.2.1 — Reemplazar `RobustScaler()`. |
| Cambiar el **número de clusters** | Sección 1.2.2 — Ajustar `n_clusters` en `KMeans()`. |
| Modificar los **porcentajes de descuento** | Sección 1.2.3 — Editar el diccionario de mapeo cluster → descuento. |
| Usar **otro algoritmo de clustering** | Sección 1.2.4 — Modificar o ampliar con otros algoritmos de `sklearn`. |
| Modificar las **consultas SQL** | Sección 2 — Editar las queries en las celdas correspondientes. |

---

## Licencia

Este proyecto está licenciado bajo la [Licencia MIT](LICENSE).
