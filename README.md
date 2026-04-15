# EDA — Contaminación Atmosférica y Salud Respiratoria en España (2011-2019)

## Descripción del Proyecto

Este proyecto realiza un **Análisis Exploratorio de Datos (EDA)** y un **contraste de hipótesis** sobre la relación entre la contaminación atmosférica y los indicadores de salud respiratoria en España durante el periodo 2011-2019. El análisis integra datos de calidad del aire de la Agencia Europea de Medio Ambiente (EEA) con datasets de salud pública de Eurostat/INE.

## Estructura del Proyecto

```
src/
├── data/
│   ├── context_clean/          # Datasets de contexto limpios (salud, población, estaciones)
│   ├── graficos/               # Visualizaciones y gráficos generados
│   ├── integrated/             # Datasets integrados (contaminantes + estaciones unidos)
│   ├── pollutants/
│   │   ├── raw/                # CSV crudos convertidos desde Parquet de la EEA
│   │   └── processed/          # Parquet limpios por contaminante
│   ├── hospitalizations_spain.csv
│   ├── mortality_respiratory_spain.csv
│   ├── population_spain.csv
│   ├── stations_metadata_spain.csv
│   └── ParquetFilesUrls_*.csv  # URLs a los archivos Parquet de la EEA (uno por contaminante)
└── notebooks/
    ├── 01_build_dataset.ipynb
    ├── 02_pollutants.ipynb
    ├── 03_context.ipynb
    ├── 04_integration_and_merge.ipynb
    ├── 05_exploratory_data_analysis.ipynb
    └── 06_hypothesis_testing.ipynb
```

## Pipeline de Notebooks

El proyecto sigue un pipeline secuencial de 6 etapas:

### 01 - Construcción del Dataset (`01_build_dataset.ipynb`)

Descarga, convierte y construye el dataset de contaminantes a partir de los archivos Parquet oficiales de la EEA.

**Operaciones principales:**

- Carga los CSV que contienen las URLs de los archivos Parquet de la EEA
- Descarga los archivos Parquet desde los servidores de la EEA
- Convierte cada Parquet a CSV para evitar problemas de compatibilidad
- Concatena cientos de CSV por cada contaminante
- Genera archivos Parquet limpios, planos y homogéneos (uno por contaminante)
- Procesa cada contaminante por separado para no saturar la RAM

**Contaminantes procesados:** PM2.5, PM10, NO₂, O₃, SO₂, CO, C₆H₆
**Nota:** El Carbono Negro (BC) se excluye por no disponer de datos para el periodo 2011-2019

---

### 02 - Contaminantes (`02_pollutants.ipynb`)

Reducción y exploratorio inicial de los datasets de contaminantes.

**Operaciones principales:**

- Carga Parquet grandes por row groups para gestionar la memoria
- Filtra el rango temporal a 2011-2019 (periodo común entre todos los datasets)
- Convierte mediciones horarias a agregación diaria
- Reduce el volumen extremo de datos a datasets manejables y comparables

---

### 03 - Contexto (`03_context.ipynb`)

Exploratorio inicial y limpieza de los datasets de contexto.

**Datasets procesados:**

- **Hospitalizaciones:** Ingresos hospitalarios por enfermedades respiratorias en España (ICD-10 J00-J99)
- **Mortalidad:** Datos de mortalidad por enfermedades respiratorias
- **Población:** Población por provincia/comunidad (para calcular tasas por cada 100.000 habitantes)
- **Estaciones:** Metadatos de las estaciones de calidad del aire (ubicación, tipo, zona, etc.)

**Operaciones principales:**

- Exploratorio inicial (shape, tipos, valores nulos, duplicados)
- Reducción de columnas a los campos esenciales
- Limpieza y estandarización de datos
- No se realizan merges en esta fase

---

### 04 - Integración y Merge (`04_integration_and_merge.ipynb`)

Integra los datos de contaminantes con los metadatos de estaciones.

**Operaciones principales:**

- Extrae y normaliza el Código Nacional (NatCode) de la columna `Samplingpoint`
- Normaliza y limpia el dataframe de estaciones para crear `df_stations_unique`
- Filtra contaminantes para asegurar correspondencia con estaciones válidas
- Realiza el merge final contaminante-estación con integridad many_to_one
- Construye un dataframe integrado (`df_all`) con todos los contaminantes

**Resultado:** Dataset unificado listo para el EDA del Notebook 05

---

### 05 - Análisis Exploratorio (`05_exploratory_data_analysis.ipynb`)

EDA integrado completo utilizando el dataframe unificado.

**Análisis realizados:**

1. EDA general por contaminante (distribuciones, boxplots, outliers)
2. Comparaciones por tipo de estación (urbana, tráfico, fondo) — **H2**
3. Comparaciones costa vs interior — **H3**
4. Series temporales y estacionalidad, especialmente O₃ — **H4**
5. Análisis espacial y mapas
6. Integración con hospitalizaciones y mortalidad — **H1**

**Limpieza de datos:** Eliminación de valores de concentración negativos (~6% del total, atribuidos a ruido instrumental/errores de calibración)

---

### 06 - Contraste de Hipótesis (`06_hypothesis_testing.ipynb`)

Evaluación de las cuatro hipótesis del proyecto utilizando los datasets limpios y las visualizaciones generadas.

## Hipótesis

### H1 — La contaminación atmosférica aumenta el impacto en la salud respiratoria

Se espera que los años con mayores niveles de contaminación presenten un aumento en hospitalizaciones y mortalidad respiratoria.

### H2 — Las zonas costeras presentan menor contaminación que las zonas de interior

La dispersión atmosférica suele ser mejor en áreas cercanas al mar, por lo que se anticipan niveles más bajos en estaciones costeras.

### H3 — Las estaciones urbanas y de tráfico presentan mayores niveles de contaminación

Las estaciones clasificadas como tráfico y urbano deberían registrar concentraciones más altas que las estaciones rurales o de fondo.

### H4 — El ozono (O₃) aumenta en verano debido a procesos fotoquímicos

El ozono troposférico depende de la radiación solar, por lo que se espera observar un patrón estacional claro con valores más altos en verano.

## Fuentes de Datos

| Dataset                          | Fuente                                  | Descripción                                                                        |
| -------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------------- |
| Concentraciones de contaminantes | Agencia Europea de Medio Ambiente (EEA) | Mediciones horarias de estaciones de calidad del aire en toda España               |
| Hospitalizaciones                | Eurostat                                | Altas hospitalarias anuales por diagnóstico, pacientes ingresados (ICD-10 J00-J99) |
| Mortalidad                       | Eurostat/INE                            | Datos anuales de mortalidad por enfermedades respiratorias                          |
| Población                       | Eurostat/INE                            | Población anual por región                                                        |
| Metadatos de estaciones          | EEA                                     | Ubicación, tipo, zona y detalles operativos de cada estación de monitorización   |

## Periodo Temporal

**2011-2019** — Seleccionado como el periodo de solapamiento común entre todos los datasets:

- Hospitalizaciones: 2000-2019
- Mortalidad: 2011-2024
- Población: 2000-2019

## Tecnologías

- **Python 3.11**
- **pandas** — Manipulación y análisis de datos
- **numpy** — Operaciones numéricas
- **matplotlib** y **seaborn** — Visualización de datos
- **pyarrow** y **fastparquet** — Manejo de archivos Parquet
- **requests** — Descargas HTTP desde la EEA
- **tqdm** — Barras de progreso

## Decisiones Técnicas Clave

1. **Carga por row groups:** Los Parquet grandes se cargan en fragmentos para evitar saturar la RAM
2. **Procesamiento por contaminante:** Cada contaminante se procesa de forma independiente para gestionar la memoria
3. **Conversión Parquet → CSV → Parquet:** Los Parquet originales de la EEA se convierten para garantizar compatibilidad con pandas
4. **Extracción del NatCode:** Extracción mediante regex de códigos de 8 dígitos desde los identificadores complejos de `Samplingpoint`
5. **Estandarización de unidades:** CO convertido de mg/m³ a µg/m³ (×1000) para permitir comparaciones entre contaminantes
6. **Eliminación de valores negativos:** ~306K registros con concentración negativa (~6%) eliminados por considerarse artefactos instrumentales

## Archivos generados

- `data/pollutants/processed/*.parquet` — Datasets limpios de contaminantes (uno por contaminante)
- `data/context_clean/*.csv` — Datasets de contexto limpios
- `data/integrated/df_pollutants_stations.parquet` — Dataset completamente unido
- `data/graficos/` — Visualizaciones generadas

## Notas

- El **Notebook 01** solo debe ejecutarse una vez (produce los datasets base)
- Los notebooks posteriores trabajan directamente con los Parquet limpios generados
- Todos los datasets de salud están a nivel nacional (agregado España), no territorial/provincial
- El proyecto se centra en enfermedades respiratorias (código ICD-10 J: J00-J99)
