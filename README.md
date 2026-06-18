# ✈️ Clustering-en-la-Segmentacion-Estrategica-de-Clientes


Proyecto de clustering no supervisado para identificar segmentos de clientes de una aerolínea con base en su comportamiento de vuelo y uso de millas, con el objetivo de diseñar estrategias de fidelización personalizadas.

Problemática: La aerolínea aplicaba una estrategia de marketing genérica que fallaba en reconocer y recompensar las diferentes formas de lealtad de sus viajeros.

---

## 📂 Dataset

EastWest Airlines — disponible en Kaggle: https://www.kaggle.com/datasets/singhnproud77/eastwestairlines-heirarchical-clustering

| Variable | Descripción |
|---|---|
| Balance | Millas acumuladas en la cuenta |
| Flight_trans_12 | Número de vuelos en los últimos 12 meses |
| Flight_miles_12mo | Millas voladas en los últimos 12 meses |
| Bonus_miles | Millas obtenidas por actividades distintas a vuelos |
| Bonus_trans | Número de transacciones de bonificación |
| Days_since_enroll | Días desde que el cliente se inscribió al programa |
| cc1_miles, cc2_miles, cc3_miles | Millas acumuladas por tipo de tarjeta de crédito |
| Qual_miles | Millas calificadas |
| Award | Indicador de uso de premios |

---

## 🛠️ Metodología

### 1. Análisis Exploratorio (EDA)
- La columna Balance presenta gran asimetría: la mayoría de clientes acumula pocas millas, mientras una minoría acumula cantidades muy superiores.
- Bonus_miles muestra una relación inversa: hay clientes con muchos bonus miles a pesar de no volar frecuentemente, lo que indica que obtienen puntos por otras vías (tarjetas de crédito, socios comerciales).

### 2. Feature Engineering
Se crearon tres variables derivadas para enriquecer el análisis:

| Variable | Fórmula | Interpretación |
|---|---|---|
| total_miles | cc1 + cc2 + cc3 | Millas totales acumuladas por tarjetas |
| flight_performance | Balance / Days_since_enroll | Ritmo de acumulación de millas por día |
| millas_promedio | Flight_miles_12mo / Flight_trans_12 | Distancia promedio por vuelo |

### 3. Preprocesamiento
- Transformación logarítmica (log1p) para reducir el efecto de outliers en variables continuas
- Estandarización con StandardScaler

### 4. Reducción de Dimensionalidad
Se aplicó t-SNE previo a la clusterización para explorar la estructura de los datos en 2D. Se eligió sobre PCA por su mayor eficacia preservando densidad y separación local, lo que permitió identificar grupos visualmente antes de modelar.

### 5. Modelado — DBSCAN
Se eligió DBSCAN sobre otros algoritmos por su capacidad para detectar clusters de formas irregulares y tratar outliers como ruido de forma explícita.
- Selección de hiperparámetros mediante la curva de distancia k (k-distance plot)
- Parámetros finales: eps=1.6, min_samples=35

### 6. Visualización
- Proyección final de clusters con t-SNE (visualización) y PCA (comparación)

---

## 📊 Resultados — Clusters Identificados

El modelo identificó 4 segmentos de clientes más un grupo de ruido:

| Cluster | Nombre | Perfil | Estrategia |
|---|---|---|---|
| 0 | 🧊 Cliente Básico o Inactivo | Cero millas promedio, sin transacciones de vuelo | Reactivación con ofertas de vuelo agresivas |
| 1 | 💎 Viajero de Alto Valor y Status | Máximo balance, alta actividad de vuelo y antigüedad | Retención, reconocimiento y programas exclusivos |
| 2 | 💳 Acumulador de Alto Status Inactivo | Sin actividad de vuelo, antigüedad alta, alto potencial financiero | Incentivos para canjear millas en vuelos y upgrades |
| 3 | 🛫 Viajero Ocasional y Eficiente | Frecuencia de vuelo moderada, milla promedio alta | Promover upgrade de tarjeta a nivel superior |
| -1 | ⚠️ Ruido | Alto balance y alta actividad — posibles Super-VIPs o anomalías | Inspección manual |

---

## 🚀 Cómo ejecutar

### 1. Clona el repositorio

git clone https://github.com/RichLD/Clustering-en-la-Segmentaci-n-Estrat-gica-de-Clientes.git
cd Clustering-en-la-Segmentaci-n-Estrat-gica-de-Clientes

### 2. Instala las dependencias

pip install -r requirements.txt

### 3. Abre el notebook

jupyter notebook "Clustering en Segmentación de Clientes.ipynb"

> El dataset se descarga automáticamente desde Kaggle con kagglehub. Necesitas tener configuradas tus credenciales de Kaggle en ~/.kaggle/kaggle.json.

---

## 📦 Dependencias

pandas, numpy, matplotlib, seaborn, scikit-learn, plotly, kagglehub

---

## 🗂️ Estructura del proyecto

Clustering en Segmentación de Clientes.ipynb
requirements.txt
README.md
dbscan_model.pkl

---

## 👤 Autor

**Ricardo López Degollado**
LinkedIn: https://linkedin.com/in/tu-perfil · GitHub: https://github.com/RichLD
