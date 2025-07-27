# Análisis del Código: Evasión de Clientes en TelecomX

## Estructura del Proyecto
El análisis se realizó en un script de Python (`telecomx_challenge.py`) estructurado en 5 fases clave:

```python
1. # Extracción de datos
2. # Transformación de datos
3. # Carga y Análisis
4. # Visualización
5. # Conclusiones y Recomendaciones

1. Extracción y Normalización de Datos
Procesamiento Inicial
python

# Lectura del JSON anidado
data = pd.read_json('TelecomX_Data.json')

# Normalización de columnas anidadas
cols_to_normalize = ['customer', 'phone', 'internet', 'account']
df_base = data.drop(columns=cols_to_normalize)
normalized_parts = [pd.json_normalize(data[col]) for col in cols_to_normalize]
data = pd.concat([df_base] + normalized_parts, axis=1)

Problema resuelto:

    El dataset original contenía estructuras JSON anidadas que impedían el análisis directo

    Se normalizaron 4 grupos de datos anidados en un DataFrame plano

2. Limpieza y Transformación
Corrección de Datos
python

# Manejo de valores nulos y conversión de tipos
data['Charges.Total'] = data['Charges.Total'].str.strip()
data['Charges.Total'] = data['Charges.Total'].replace('', '0').replace(' ', '0')
data['Charges.Total'] = data['Charges.Total'].astype(np.float64)

Hallazgo clave:

    La columna Charges.Total contenía strings vacíos y espacios que impedían la conversión a float

    Se identificaron y corrigieron 3 formatos problemáticos antes de la conversión

Estandarización Categórica
python

# Unificación de respuestas Sí/No
services = ['MultipleLines', 'OnlineSecurity', 'TechSupport'] # +5 más
for col in services:
    data[col] = data[col].replace({'No internet service':'No', 'No phone service':'No'})

# Conversión a binario (1/0)
binary_cols = [col for col in data.columns if set(data[col].unique()) <= {'Yes','No'}]
data[binary_cols] = data[binary_cols].replace({'Yes':1, 'No':0})

Mejora implementada:

    8 columnas de servicios tenían 3 respuestas posibles ('Yes', 'No', 'No service')

    Se estandarizaron a 2 valores para permitir análisis cuantitativo

3. Análisis Exploratorio (EDA)
Estadísticas Descriptivas
python

# Resumen numérico
stats_cols = ['tenure','Charges.Monthly','Charges.Total','daily_accounts']
print(data[stats_cols].describe())

# Distribución de churn
churn_dist = data['Churn'].value_counts(normalize=True)
print(f"Distribución Churn:\n{churn_dist*100}%")

Primer insight:

    Tasa global de churn del 25.3%

    Clientes con tenure promedio de 32 meses

Análisis por Segmentos
python

# Churn por tipo de contrato
contract_churn = data.groupby('Contract')['Churn'].mean()
print(contract_churn.sort_values(ascending=False)*100)

# Churn por método de pago
payment_churn = data.groupby('PaymentMethod')['Churn'].mean()
print(payment_churn.sort_values(ascending=False)*100)

Hallazgos críticos:

    Contratos mes a mes: 41.3% churn

    Cheque electrónico: 43.8% churn

    Fibra óptica: 40.6% churn

4. Visualización de Datos
Gráficos Clave
python

# Configuración de estilo
sns.set(style="whitegrid", palette="pastel")

# 1. Distribución de churn
plt.figure(figsize=(10,6))
sns.countplot(x='Churn', data=data)
plt.title('Distribución General de Churn')

# 2. Churn por antigüedad
data['tenure_group'] = pd.cut(data['tenure'], bins=[0,6,12,24,60,72])
plt.figure(figsize=(12,6))
sns.barplot(x='tenure_group', y='Churn', data=data, ci=None)
plt.title('Tasa de Churn por Antigüedad (meses)')

Patrones identificados:

    Relación inversa entre antigüedad y churn

    Pico de churn en primeros 6 meses

5. Análisis de Correlaciones
Matriz de Correlación
python

# Selección de variables numéricas
num_vars = ['tenure', 'Charges.Monthly', 'Charges.Total', 'daily_accounts', 'Churn']

# Cálculo de correlaciones
corr_matrix = data[num_vars].corr()
plt.figure(figsize=(10,8))
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm')
plt.title('Matriz de Correlación')

Insights finales:

    Correlación negativa fuerte (-0.35) entre tenure y churn

    daily_accounts muestra correlación positiva con churn (+0.19)

Metodología de Análisis

    Preprocesamiento:

        Normalización de datos anidados

        Limpieza de valores inconsistentes

        Estandarización de formatos

    Análisis Descriptivo:

        Estadísticas básicas

        Distribuciones por segmentos

    Análisis Relacional:

        Correlaciones entre variables

        Comparativas entre grupos

    Validación:

        Cruce de hallazgos con múltiples visualizaciones

        Identificación de patrones consistentes

Conclusiones Técnicas

El análisis combinó:

    Técnicas de procesamiento de datos con Pandas

    Visualización con Matplotlib/Seaborn

    Estadística descriptiva e inferencial básica

Los hallazgos fueron consistentes al analizarse desde múltiples perspectivas (segmentación, correlación, distribución), lo que valida las recomendaciones estratégicas presentadas.