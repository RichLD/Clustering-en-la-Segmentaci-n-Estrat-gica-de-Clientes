# ✈️ Clustering en la Segmentación Estratégica de Clientes — Sector Aeronáutico

## De qué se trata esto

Este proyecto nace de una pregunta simple: ¿dos clientes con el mismo balance de millas son realmente el mismo tipo de cliente? Trabajando con datos de un programa de lealtad de aerolínea, me encontré con que no — hay clientes con balances casi idénticos (entre $90K y $116K millas) que se comportan de forma completamente opuesta: unos vuelan varias veces al año, otros casi nunca suben a un avión y acumulan sus millas por gastar con la tarjeta de crédito co-branded. Tratarlos como el mismo segmento "de alto valor" sería, sencillamente, un error.

Elegí el sector aeronáutico a propósito. El comportamiento de un viajero frecuente es de los más difíciles de modelar que existen: mezcla hábitos de consumo financiero, frecuencia real de viaje, y acumulación de puntos por canales que no tienen nada que ver con volar. Es justo ese tipo de complejidad la que hace que un clustering bien hecho (y uno mal hecho) se noten tanto.

## El problema de negocio

La mayoría de programas de lealtad segmenta por balance o antigüedad — dos métricas fáciles de calcular pero que esconden diferencias de comportamiento que sí importan a la hora de decidir qué campaña enviarle a quién. La idea aquí fue ir más allá de esas dos variables e identificar grupos que reflejen cómo se comporta la gente de verdad, para poder diseñar estrategias distintas según el tipo de relación que cada segmento tiene con la aerolínea.

## Los datos

- **Fuente:** [East-West Airlines en Kaggle](https://www.kaggle.com/datasets/singhnproud77/eastwestairlines-heirarchical-clustering) — un dataset público de uso académico
- **Tamaño:** 3,999 clientes, 12 variables

Vale aclarar esto desde ahora: es un dataset académico, no de una aerolínea real ni de un proyecto profesional. Lo digo sin rodeos porque prefiero que la interpretación de los resultados sea honesta desde el inicio.

| Variable | Qué significa |
|---|---|
| Balance | Millas acumuladas en la cuenta |
| Qual_miles | Millas de calificación — 94% de los clientes tiene cero, así que es una variable bastante particular (zero-inflated) |
| cc1_miles, cc2_miles, cc3_miles | Millas por tipo de tarjeta de crédito asociada — cc2 y cc3 resultaron casi constantes (más del 99% de los clientes cae en la misma categoría) |
| Bonus_miles | Millas por actividades que no son volar (tarjetas, socios comerciales) |
| Bonus_trans | Transacciones de bonificación |
| Flight_miles_12mo | Millas voladas en los últimos 12 meses |
| Flight_trans_12 | Vuelos en los últimos 12 meses |
| Days_since_enroll | Antigüedad en el programa |
| Award | Si canjeó o no un premio |

También construí tres variables propias:

| Variable | Cómo se calcula | Para qué sirve |
|---|---|---|
| total_miles | cc1 + cc2 + cc3 | Millas totales por tarjetas (aunque su utilidad quedó limitada por lo constante de cc2/cc3) |
| flight_performance | Balance / Days_since_enroll | Qué tan rápido acumula millas un cliente por día |
| millas_promedio | Flight_miles_12mo / Flight_trans_12 | Distancia promedio por vuelo (0 si no voló) |

## Cómo llegué a la metodología final

Esta sección probablemente sea la parte más útil del README, porque no fue un camino recto — hubo un par de decisiones que tomé al inicio que tuve que revisar y corregir después de pensarlas más a fondo (y de que me hicieran las preguntas correctas sobre ellas).

**Outliers: nada de reglas genéricas.** Al principio es tentador aplicar una sola regla (tipo IQR) a todo el dataset y ya. Pero al meterme más a fondo con cada variable me di cuenta de que eso hubiera sido un error: `Qual_miles` tiene 94% de ceros, así que sus valores "atípicos" en realidad son el segmento de viajeros con estatus élite, no ruido que limpiar. Terminé creando una bandera (`has_qual_miles`) para no perder esa señal. Por otro lado, `cc2_miles` y `cc3_miles` casi no varían entre clientes (más del 99% cae en la misma categoría) — los dejé sin tocar, y al final confirmé que efectivamente casi no aportaban nada al modelo (effect size de 0.002-0.003, prácticamente cero). Al resto de variables con sesgo real (Balance, Bonus_miles, millas de vuelo) les apliqué log1p.

**Por qué t-SNE y no PCA.** Antes de decidir, calculé la correlación entre variables — la más alta fue 0.40, lo cual significa que PCA no iba a lograr comprimir mucha varianza en pocos componentes. Con eso en mano, t-SNE tenía más sentido.

**Calibrar t-SNE en serio.** La primera vez que corrí t-SNE usé la perplejidad por default y el resultado se veía lleno de "grumos" pequeños — un clásico síntoma de sobreajuste. Hice un barrido (5, 15, 30, 50, 80) y terminé usando 50, que es más o menos la raíz cuadrada del tamaño de la muestra (una regla práctica bastante usada). Importante: esto lo usé solo para visualizar, el clustering en sí lo corrí sobre el espacio completo de variables, no sobre las 2 dimensiones de t-SNE.

**Probé tres algoritmos, no me quedé con el primero que funcionó.**

Empecé con DBSCAN porque tiene sentido para detectar outliers de forma explícita. El problema: al principio ajusté `eps` buscando que me diera más clusters (algo así como forzar el resultado que yo quería ver), sin fijarme en lo que el gráfico de codo realmente indicaba. Cuando lo hice bien — usando el valor de `eps` que marca el codo real —, DBSCAN colapsó a un solo cluster (98.4% de los clientes) más un 1.6% de ruido. En otras palabras: la estructura de densidad de estos datos no soporta varios grupos naturales. Fue un resultado incómodo de aceptar al principio, pero es el correcto.

Ahí fue cuando cambié de herramienta. Para lograr una segmentación de negocio con un número específico de grupos, KMeans y GMM son los algoritmos correctos — a ellos sí les puedes decir "quiero 3 grupos" de forma explícita, sin forzar nada. Corrí ambos con k=3 y los comparé:

| Modelo | Silhouette | Effect size grande en |
|---|---|---|
| KMeans | 0.257 | 5 de 6 variables |
| GMM | 0.254 | 5 de 6 variables |

Quedaron prácticamente empatados. Me quedé con KMeans no porque sea estadísticamente mejor —no lo es, casi es idéntico a GMM— sino porque es más simple de explicar e implementar en un contexto de negocio real.

**Validar que las diferencias fueran reales, no solo "significativas".** Con casi 4,000 clientes, casi cualquier diferencia sale estadísticamente significativa aunque sea mínima. Por eso además del p-value calculé el tamaño de efecto (epsilon cuadrada) de cada variable entre los 3 clusters finales:

| Variable | p-value | Effect size (ε²) |
|---|---|---|
| Flight_trans_12 | <0.0001 | 0.912 |
| millas_promedio | <0.0001 | 0.910 |
| Bonus_miles | <0.0001 | 0.552 |
| cc1_miles | <0.0001 | 0.497 |
| Balance | <0.0001 | 0.354 |
| Days_since_enroll | <0.0001 | 0.048 |

Cinco de seis variables con efecto grande — la segmentación está realmente sostenida por comportamiento de vuelo y acumulación de millas, no por cuánto tiempo lleva el cliente inscrito (esa variable casi no diferencia a nadie, y prefiero decirlo así de claro en vez de esconderlo).

## Los 3 segmentos que salieron

| Segmento | Tamaño | Balance | Vuelos/año | En resumen |
|---|---|---|---|---|
| **Bajo compromiso** | 1,573 (39%) | ~$26.7K | 0.05 | Poca actividad en general, el segmento más "dormido" |
| **Viajeros activos** | 1,215 (30%) | ~$116K | 4.46 | El cliente que uno se imagina cuando piensa en "cliente frecuente" de aerolínea |
| **Acumuladores de tarjeta** | 1,211 (30%) | ~$91.7K | ~0.00 | Balance parecido al anterior, pero casi no vuelan — su relación con la aerolínea es más financiera que de viaje |

Lo que más me gustó de este resultado es justo la comparación entre los últimos dos: balances casi idénticos, comportamientos opuestos. Ahí está el valor de hacer esto bien.

## Qué haría una aerolínea con esto

A "Viajeros activos" y "Acumuladores de tarjeta" no les convendría enviarles la misma campaña aunque ambos parezcan "clientes de alto valor" en el balance. Al primero se le retiene con beneficios de vuelo — upgrades, millas dobles, prioridad de abordaje. Al segundo, con productos financieros o beneficios ligados a la tarjeta, porque su vínculo real con la marca pasa por ahí, no por volar.

El segmento de "Bajo compromiso" (39% de la base, nada despreciable) es un buen candidato para probar campañas de reactivación de bajo costo antes de tirarle descuentos agresivos que quizás ni necesita para reengancharse.

## Lo que le falta a esto (y me parece justo decirlo)

- Es un dataset académico. No pasó por revisión de un equipo de negocio real, así que trátalo como una demostración de proceso, no como una consultoría terminada
- Es una foto fija en el tiempo. Ver cómo los clientes migran entre segmentos a lo largo de los meses sería el siguiente paso lógico
- No probé clustering jerárquico ni HDBSCAN, que podrían encontrar sub-segmentos dentro de "Viajeros activos" (por ejemplo, frecuencia alta vs. media)

## Cómo correrlo tú mismo

```bash
git clone https://github.com/RichLD/Clustering-en-la-Segmentaci-n-Estrat-gica-de-Clientes.git
cd Clustering-en-la-Segmentaci-n-Estrat-gica-de-Clientes
pip install -r requirements.txt
jupyter notebook "Clustering en Segmentación de Clientes.ipynb"
```

El dataset se descarga automáticamente con `kagglehub` desde la primera celda (necesitas tus credenciales de Kaggle en `~/.kaggle/kaggle.json`), o lo puedes bajar manualmente desde el link de arriba.

**Dependencias:** pandas, numpy, scikit-learn, scipy, matplotlib, seaborn, plotly, kagglehub

**Estructura del repo:**
```
Clustering en Segmentación de Clientes.ipynb
requirements.txt
README.md
kmeans_model.pkl
```


## 👤 Autor

**Ricardo López Degollado**
LinkedIn: https://www.linkedin.com/in/ricardo-l%C3%B3pez-degollado-a6a968203/
GitHub: https://github.com/RichLD
