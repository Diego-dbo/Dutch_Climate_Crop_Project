# Dutch Wheat Production and Meteorological Data Analysis

## Fao_Table - Task Log

### Data Preparation

- **Data Collection**  
  **Source:** FAO  
  **Data:** Downloaded data on wheat production in the Netherlands for all available periods from the following link: [FAO Data](https://www.fao.org/faostat/en/#data/QCL).

### Data Processing

- **Data Cleaning and Transformation using SQL and BigQuery**

  - **Removal of Unnecessary Columns**  
    ```sql
    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN domain_code;

    ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    DROP COLUMN item;

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

  - **Conversion of the `value` column from integer to float64**  
    ```sql
    ALTER TABLE 
    `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    ALTER COLUMN value SET DATA TYPE FLOAT64;
    ```

  - **Conversion of measurement units from 100 grams per hectare (100 g/ha) to tons per hectare (ton/ha), rounded to two decimal places**  
    ```sql
    UPDATE 
    `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    SET 
    value = ROUND(value * 0.0001, 2)
    WHERE 
    unit = '100 g/ha';
    ```
  - **Replacing the unit of 100 g/ha with ton/ha in the `unit` column**  
    ```sql
    UPDATE 
    `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    SET 
    unit = 'ton/ha'
    WHERE 
    unit = '100 g/ha';
    ```
  - **Simplifying long descriptions**  
    ```sql
    UPDATE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    SET flag_description = 'missing'
    WHERE flag_description = 'missing value data cannot exist, not applicable';
    ```
  - **Missing Values Check in All Columns**  
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
  - **Duplicate Data Handling**  
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
  - **Consistency Check for Values**  
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

### Data Analysis

   - **Temporal analysis of the elements: Yield, production, and area harvested (1961-2022)**  
   ```sql
    SELECT element, year, value 
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    ORDER BY year ASC;
    ```
   - **Year Comparison of Wheat Yield, Production, and Harvested Area with Percentage Change from First to Last Year**
   ```sql
    WITH data_comparison AS (
    SELECT 
    element,
    year,
    value,
    FIRST_VALUE(value) OVER (PARTITION BY element ORDER BY year ASC) AS first_year_value,
    LAST_VALUE(value) OVER (PARTITION BY element ORDER BY year ASC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_year_value
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
  
      )

    SELECT 
    element,
    MIN(year) AS first_year,
    MAX(year) AS last_year,
    first_year_value,
    last_year_value,
    (last_year_value - first_year_value) AS difference,
    ((last_year_value - first_year_value) / first_year_value) * 100 AS percentage_change
    FROM data_comparison
    GROUP BY element, first_year_value, last_year_value
    ORDER BY element;
  ```
- **Correlation Analysis between Wheat Yield, Area Harvested, and Production (1961-2022)**
  ```sql
  WITH data AS (
  SELECT 
    year,
    MAX(CASE WHEN element = 'yield' THEN value END) AS yield_value,
    MAX(CASE WHEN element = 'area harvested' THEN value END) AS area_harvested_value,
    MAX(CASE WHEN element = 'production' THEN value END) AS production_value
  FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
  GROUP BY year
  )

  SELECT 
  'Yield vs Area Harvested' AS correlation_between,
  CORR(yield_value, area_harvested_value) AS correlation_value
  FROM data

  UNION ALL

  SELECT 
  'Yield vs Production' AS correlation_between,
  CORR(yield_value, production_value) AS correlation_value
  FROM data

  UNION ALL

  SELECT 
  'Production vs Area Harvested' AS correlation_between,
  CORR(production_value, area_harvested_value) AS correlation_value
  FROM data;
  ```
  - **Top 3 Increases and Decreases in Yield, Production, and Area Harvested (Yearly Comparison)**
    ```sql
    WITH changes AS (
    SELECT 
    year,
    element,
    value,
    LAG(value) OVER (PARTITION BY element ORDER BY year) AS previous_year_value,
    ((value - LAG(value) OVER (PARTITION BY element ORDER BY year)) / LAG(value) OVER (PARTITION BY element ORDER BY year)) * 100 AS percent_change
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    WHERE element IN ('production', 'yield', 'area harvested')
    )

    , ranked_changes AS (
    SELECT 
    element,
    year,
    value,
    percent_change,
    ROW_NUMBER() OVER (PARTITION BY element ORDER BY percent_change DESC) AS rank_increase,
    ROW_NUMBER() OVER (PARTITION BY element ORDER BY percent_change ASC) AS rank_decrease
    FROM changes
    WHERE percent_change IS NOT NULL
    )

    -- Select top 3 increases and top 3 decreases for each element
    SELECT 
    element,
  
    -- Top 3 Increases
    MAX(CASE WHEN rank_increase = 1 THEN year END) AS year_top_increase_1,
    MAX(CASE WHEN rank_increase = 1 THEN value END) AS value_top_increase_1,
    MAX(CASE WHEN rank_increase = 1 THEN percent_change END) AS percent_change_top_increase_1,
  
    MAX(CASE WHEN rank_increase = 2 THEN year END) AS year_top_increase_2,
    MAX(CASE WHEN rank_increase = 2 THEN value END) AS value_top_increase_2,
    MAX(CASE WHEN rank_increase = 2 THEN percent_change END) AS percent_change_top_increase_2,

    MAX(CASE WHEN rank_increase = 3 THEN year END) AS year_top_increase_3,
 ```




---

