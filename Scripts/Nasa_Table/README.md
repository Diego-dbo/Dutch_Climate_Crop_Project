# Dutch Agricultural and Meteorological Data Analysis

Nasa_Table - Registro de Tarefas

## Data Preparation

Coleta dos Dados  
Fonte: NASA  
Dados: Baixei os dados meteorológicos através do link [NASA Power Data Access Viewer](https://power.larc.nasa.gov/data-access-viewer/)

**Nota**: Devido às limitações da ferramenta "Rectangle Sketch Tool", não foi possível selecionar apenas a área dos Países Baixos. Como resultado, os dados incluem regiões dos países vizinhos. Para resolver esse problema, utilizei o software QGIS para a exclusão dos dados não requeridos na etapa de processamento de dados.

## Data Processing

Limpeza e Transformação dos Dados Usando QGIS  
- Seleção dos dados provenientes apenas dos Países Baixos.
- Importação do Arquivo CSV: Carreguei o arquivo CSV contendo os dados meteorológicos no QGIS.
- Adição do Shapefile da Holanda: Importei um shapefile com as delimitações dos Países Baixos no QGIS para filtrar as áreas específicas do país.
- Processo de Interseção:
  - Configuração das Camadas: Configurei as colunas de latitude (LAT) e longitude (LON) no CSV para posicionar corretamente os dados geográficos.
  - Interseção das Camadas: Usei a ferramenta de interseção do QGIS para cruzar os dados meteorológicos com o shapefile da Holanda, eliminando os dados referentes às regiões fora dos Países Baixos.
- Verificação e Salvamento:
  - Salvamento: Salvei o resultado final em um novo arquivo CSV contendo apenas os dados meteorológicos referentes aos Países Baixos.
  - Verificação Visual: Abri e visualizei os dados do novo arquivo plotados no QGIS para garantir que os dados filtrados correspondiam apenas aos Países Baixos.

Limpeza e Transformação dos Dados Usando Microsoft Excel  
- Padronização e Limpeza de Dados:
  - Remoção do Cabeçalho: Excluí o cabeçalho desnecessário.
  - Substituições: Troquei parênteses `()` por underscores `_` e removi espaços e aspas.

Upload para BigQuery  
- Realizei o upload do arquivo para o BigQuery com a opção de reconhecer o schema de forma automática.

Limpeza e Transformação de Dados usando SQL e BigQuery  

- Padronizacao dos nomes das colunas para letras minuculas.

```sql
ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN January TO january;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN February TO february;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN March TO march;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN April TO april;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN May TO may;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN June TO june;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN July TO july;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN August TO august;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN September TO september;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN October TO october;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN November TO november;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN December TO december;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN Annual_temp TO annual_temp;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN Year TO year;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN Latitude TO latitude;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN Longitude TO longitude;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN Name TO name;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN Variable TO variable;

ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
RENAME COLUMN Unit TO unit;

```
- Conversão de Tipos de Dados:
  ```sql
  CREATE OR REPLACE TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table AS
  SELECT
      SAFE_CAST(january AS FLOAT64) AS january,
      SAFE_CAST(february AS FLOAT64) AS february,
      SAFE_CAST(march AS FLOAT64) AS march,
      SAFE_CAST(april AS FLOAT64) AS april,
      SAFE_CAST(may AS FLOAT64) AS may,
      SAFE_CAST(june AS FLOAT64) AS june,
      SAFE_CAST(july AS FLOAT64) AS july,
      SAFE_CAST(august AS FLOAT64) AS august,
      SAFE_CAST(september AS FLOAT64) AS september,
      SAFE_CAST(october AS FLOAT64) AS october,
      SAFE_CAST(november AS FLOAT64) AS november,
      SAFE_CAST(december AS FLOAT64) AS december,
      SAFE_CAST(annual_temp AS FLOAT64) AS annual_temp,
      parameter,
      year,
      latitude,
      longitude,
      id,
      name,
      source
  FROM
      my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table;
  ```

- Criação da Coluna `variable`:
  - Mapeamento:
    - `ts` = Earth Skin Temperature
    - `qv2m` = Specific Humidity
    - `rh2m` = Relative Humidity
    - `gwettop` = Surface Soil Wetness
    - `prectotcorr_sum` = Precipitation Corrected Sum
  ```sql
  UPDATE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  SET
      variable = CASE
          WHEN parameter = 'ts' THEN 'Earth Skin Temperature'
          WHEN parameter = 'qv2m' THEN 'Specific Humidity'
          WHEN parameter = 'rh2m' THEN 'Relative Humidity'
          WHEN parameter = 'gwettop' THEN 'Surface Soil Wetness'
          WHEN parameter = 'prectotcorr_sum' THEN 'Precipitation Corrected Sum'
          ELSE 'Unknown'
      END
  WHERE
      parameter IN ('ts', 'qv2m', 'rh2m', 'gwettop', 'prectotcorr_sum') OR TRUE;
  ```

- Criação da Coluna `unit`:
  ```sql
  ALTER TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  ADD COLUMN unit STRING;
  ```

  - Insercao das Unidades das Variáveis na Coluna `unit`:
  ```sql
  UPDATE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  SET unit = CASE
      WHEN variable = 'Earth Skin Temperature' THEN '(C)'
      WHEN variable = 'Specific Humidity' THEN 'at 2 Meters (g/kg)'
      WHEN variable = 'Relative Humidity' THEN 'at 2 Meters (%)'
      WHEN variable = 'Surface Soil Wetness' THEN '(1)'
      WHEN variable = 'Precipitation Corrected Sum' THEN '(mm)'
      ELSE 'Unknown'
  END
  WHERE true;
  ```

- Remoção de Colunas Não Necessárias:
  ```sql
  ALTER TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  DROP COLUMN parameter;
  
  ALTER TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  DROP COLUMN id;
  ```

## Data Validation

- Verificação de Valores Faltantes em Todas as Colunas:
  ```sql
  SELECT
      COUNT(*) AS total_rows,
      COUNTIF(latitude IS NULL) AS latitude_missing,
      COUNTIF(longitude IS NULL) AS longitude_missing,
      COUNTIF(january IS NULL) AS january_missing,
      COUNTIF(february IS NULL) AS february_missing,
      COUNTIF(march IS NULL) AS march_missing,
      COUNTIF(april IS NULL) AS april_missing,
      COUNTIF(may IS NULL) AS may_missing,
      COUNTIF(june IS NULL) AS june_missing,
      COUNTIF(july IS NULL) AS july_missing,
      COUNTIF(august IS NULL) AS august_missing,
      COUNTIF(september IS NULL) AS september_missing,
      COUNTIF(october IS NULL) AS october_missing,
      COUNTIF(november IS NULL) AS november_missing,
      COUNTIF(december IS NULL) AS december_missing,
      COUNTIF(annual_temp IS NULL) AS annual_temp_missing,
      COUNTIF(year IS NULL) AS year_missing,
      COUNTIF(name IS NULL) AS name_missing,
      COUNTIF(variable IS NULL) AS variable_missing
  FROM
      my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table;
  ```

- Verificação de Consistência das Unidades:
  ```sql
  SELECT
      variable,
      unit,
      COUNT(*) AS count
  FROM
      my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  GROUP BY
      variable,
      unit
  ORDER BY
      variable;
  ```

- Verificação de Coordenadas Geográficas:
  ```sql
  SELECT DISTINCT latitude, longitude
  FROM
      my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  WHERE
      latitude NOT BETWEEN 50 AND 54
      OR longitude NOT BETWEEN 3 AND 8;
  ```

- Verificação de Linhas Duplicadas:
  ```sql
  SELECT
      latitude,
      longitude,
      january,
      february,
      march,
      april,
      may,
      june,
      july,
      august,
      september,
      october,
      november,
      december,
      annual_temp,
      variable,
      unit,
      COUNT(*) AS duplicate_count
  FROM
      my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  GROUP BY
      latitude,
      longitude,
      january,
      february,
      march,
      april,
      may,
      june,
      july,
      august,
      september,
      october,
      november,
      december,
      annual_temp,
      variable,
      unit
  HAVING
      COUNT(*) > 1;
  
