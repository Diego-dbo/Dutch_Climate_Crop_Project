# Dutch Agricultural and Meteorological Data Analysis

## Fao_Table - Task Log

### Data Preparation

- **Data Collection**  
  **Source:** FAO  
  **Data:** Downloaded data on wheat, potato, and onion production from the following link: [FAO Data](https://www.fao.org/faostat/en/#data/QCL).

### Data Processing

- **Data Cleaning and Transformation using SQL and BigQuery**

  - **Removal of Unnecessary Columns**
    ```sql
    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN domain_code;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN area_code;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN area;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN element_code;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN item_code_cpc;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN year_code;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN flag;
    ```

  - **Standardization of Column Names to Lowercase**
    ```sql
    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    RENAME COLUMN Domain TO domain;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    RENAME COLUMN Element TO element;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    RENAME COLUMN Item TO item;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    RENAME COLUMN Year TO year;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    RENAME COLUMN Unit TO unit;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    RENAME COLUMN Value TO value;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    RENAME COLUMN Flag_Description TO flag_description;
    ```

  - **Missing Values Check**
    ```sql
    SELECT
      COUNT(*) AS total_rows,
      COUNTIF(domain IS NULL) AS domain_missing,
      COUNTIF(element IS NULL) AS element_missing,
      COUNTIF(item IS NULL) AS item_missing,
      COUNTIF(year IS NULL) AS year_missing,
      COUNTIF(unit IS NULL) AS unit_missing,
      COUNTIF(value IS NULL) AS value_missing,
      COUNTIF(flag_description IS NULL) AS flag_description_missing
    FROM
      `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`;
    ```
    -Conversão da coluna value de integer para float 64
    ```sql
    ALTER TABLE 
    `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
     ALTER COLUMN value SET DATA TYPE FLOAT64
     ```
    -Conversao dos valores de medida de 100 gramas por hectare(100g/ha)  para toneladas por hectare(ton/ha), limitando a duas casas apos a virgula.
    ```sql
    UPDATE 
    `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    SET 
    value = ROUND(value * 0.0001, 2)
    WHERE 
    unit = '100 g/ha';
    ```
    -Substitunido a unidade de 100g/ha para ton/ha na coluna unit
    ```sql
    UPDATE 
     `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    SET 
    unit = 'ton/ha'
    WHERE 
    unit = '100 g/ha';
    ```
    -Verificação de Valores Faltantes em Todas as Colunas
    ```sql
    SELECT
    COUNT(*) AS total_rows,
    COUNTIF(domain IS NULL) AS domain_missing,
    COUNTIF(element IS NULL) AS element_missing,
    COUNTIF(item IS NULL) AS item_missing,
    COUNTIF(year IS NULL) AS year_missing,
    COUNTIF(unit IS NULL) AS unit_missing,
    COUNTIF(value IS NULL) AS value_missing,
    COUNTIF(flag_description IS NULL) AS flag_description_missing
    FROM
    `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`;
     ```
   -Duplicate Data Handling
    ```sql
    SELECT
      domain,
      element,
      item,
      year,
      unit,
      value,
      flag_description,
      COUNT(*) AS duplicate_count
    FROM
      `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    GROUP BY
      domain,
      element,
      item,
      year,
      unit,
      value,
      flag_description
    HAVING
      COUNT(*) > 1;
    ```
-Verificação de Consistência de de Valores
```sql
SELECT
  element,
  MIN(value) AS min_value,
  MAX(value) AS max_value
FROM
  `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
GROUP BY
  element;
```


