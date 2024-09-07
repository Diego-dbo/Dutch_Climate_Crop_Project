# Dutch Wheat Production and Meteorological Data Analysis

Nasa_Table - Task Log

## Data Preparation

**Data Collection**  
Source: NASA  
Data: I downloaded the meteorological data through the link [NASA Power Data Access Viewer](https://power.larc.nasa.gov/data-access-viewer/).

**Note**: Due to the limitations of the "Rectangle Sketch Tool," it was not possible to select only the area of the Netherlands. As a result, the data includes regions from neighboring countries. To resolve this issue, I used QGIS software to remove unwanted data during the data processing stage.

## Data Processing

**Cleaning and Transforming the Data Using QGIS**  
- Selection of data originating only from the Netherlands.
- **CSV File Import**: Imported the CSV file containing meteorological data into QGIS.
- **Adding the Netherlands Shapefile**: Imported a shapefile with the boundaries of the Netherlands into QGIS to filter the specific areas of the country.
- **Intersection Process**:
  - **Layer Configuration**: Configured the columns for latitude (LAT) and longitude (LON) in the CSV to correctly position the geographical data.
  - **Layer Intersection**: Used the QGIS intersection tool to cross the meteorological data with the Netherlands shapefile, eliminating data from regions outside the Netherlands.
- **Verification and Saving**:
  - **Saving**: Saved the final result into a new CSV file containing only meteorological data for the Netherlands.
  - **Visual Verification**: Opened and visualized the data from the new file plotted in QGIS to ensure the filtered data corresponded only to the Netherlands.

**Cleaning and Transforming Data Using Microsoft Excel**  
- **Data Standardization and Cleaning**:
  - **Header Removal**: Removed unnecessary headers.
  - **Replacements**: Replaced parentheses `()` with underscores `_` and removed spaces and quotes.

**Upload to BigQuery**  
- Uploaded the file to BigQuery using the option to recognize the schema automatically.

**Data Cleaning and Transformation Using SQL and BigQuery**

- **Removing Unnecessary Columns**:
  ```sql
  ALTER TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  DROP COLUMN parameter;
  
  ALTER TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  DROP COLUMN id;
  ```

- **Standardizing Column Names to Lowercase**:
  ```sql
  ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
  RENAME COLUMN January TO january;

  ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
  RENAME COLUMN February TO february;

  ALTER TABLE `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
  RENAME COLUMN March TO march;
  
  -- Same process for all months and other fields
  ```

- **Data Type Conversion**:
  ```sql
  CREATE OR REPLACE TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table AS
  SELECT
      SAFE_CAST(january AS FLOAT64) AS january,
      SAFE_CAST(february AS FLOAT64) AS february,
      -- Same for all other months
      annual_temp, parameter, year, latitude, longitude, id, name, source
  FROM my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table;
  ```

- **Creating the `variable` Column**:
  - **Mapping**:
    - `ts` = Earth Skin Temperature  
    - `prectotcorr_sum` = Precipitation Corrected Sum
  ```sql
  UPDATE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  SET variable = CASE
      WHEN parameter = 'ts' THEN 'Earth Skin Temperature'
      WHEN parameter = 'prectotcorr_sum' THEN 'Precipitation Corrected Sum'
      ELSE 'Unknown'
  END
  WHERE parameter IN ('ts', 'prectotcorr_sum');
  ```

- **Creating the `unit` Column**:
  ```sql
  ALTER TABLE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  ADD COLUMN unit STRING;
  ```

  - **Inserting Units for Variables into the `unit` Column**:
  ```sql
  UPDATE my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  SET unit = CASE
      WHEN variable = 'Earth Skin Temperature' THEN '(C)'
      WHEN variable = 'Precipitation Corrected Sum' THEN '(mm)'
      ELSE 'Unknown'
  END;
  ```

- **Checking for Missing Values in All Columns**:
  ```sql
  SELECT
      COUNT(*) AS total_rows,
      COUNTIF(latitude IS NULL) AS latitude_missing,
      COUNTIF(longitude IS NULL) AS longitude_missing,
      -- Same for all other months
      COUNTIF(annual_temp IS NULL) AS annual_temp_missing
  FROM my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table;
  ```

- **Consistency Check of Units**:
  ```sql
  SELECT
  variable,
  MIN(january) AS min_january,
  MAX(january) AS max_january,
  -- Same for all other months
  MIN(annual_temp) AS min_annual_temp,
  MAX(annual_temp) AS max_annual_temp
  FROM my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  GROUP BY variable;
  ```

- **Check for Duplicate Rows**:
  ```sql
  SELECT
      latitude, longitude, january, february, -- Other months, and more fields
      COUNT(*) AS duplicate_count
  FROM my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table
  GROUP BY latitude, longitude, january, february, -- Other fields
  HAVING COUNT(*) > 1;
  ```

## Data Analysis
- **Precipitation and Surface Temperature Changes Over the Years (1984-2022)**:
  ```sql
  SELECT
    year,
    variable,
    AVG(annual_temp) AS avg_annual_temp
  FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
  GROUP BY year, variable
  ORDER BY year, variable;
  ```

- **Monthly Average Analysis of Earth Skin Temperature and Precipitation (1984-2022)**:
  ```sql
  SELECT variable, 'January' AS month, AVG(january) AS avg_value
  FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
  GROUP BY variable
  '''
- **Detecting Outliers in temperature and precipitation using Z-score**
```sql
WITH stats AS (
    -- Calculates the mean and standard deviation of annual_temp for each year and variable
    SELECT
        year,
        variable,
        AVG(annual_temp) AS mean_annual_temp,
        STDDEV(annual_temp) AS stddev_annual_temp
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
    GROUP BY year, variable
),
annual_temps_with_zscore AS (
    -- Joins the statistics with the original data and calculates the Z-score for each year and variable
    SELECT
        nt.year,
        nt.variable,
        nt.annual_temp,
        s.mean_annual_temp,
        s.stddev_annual_temp,
        (nt.annual_temp - s.mean_annual_temp) / s.stddev_annual_temp AS zscore
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table` nt
    JOIN stats s
    ON nt.year = s.year AND nt.variable = s.variable
)
SELECT
    year,
    variable,
    annual_temp,
    zscore
FROM annual_temps_with_zscore
WHERE ABS(zscore) > 3 -- Defines as an outlier any value whose absolute Z-score is greater than 3
ORDER BY year, variable;

```
- **Integrated Analysis of Wheat Yield and Climate Variables (1984-2022)**
  - **A new integrated table was created using an INNER JOIN between the FAO table (filtered to include only the yield element) and a NASA sub-table that already contained annual averages for temperature and precipitation. This allows for more streamlined queries and analysis, focusing on the relationship between wheat yield and key climate variables (temperature and precipitation) from 1984 to 2022.**

```sql
WITH nasa_data AS (
    -- Filtra os dados da tabela NASA para obter as médias anuais de temperatura e precipitação entre 1984 e 2022
    SELECT
        year,
        MAX(CASE WHEN variable = 'Earth Skin Temperature' THEN avg_annual_value END) AS avg_temperature,
        MAX(CASE WHEN variable = 'Precipitation Corrected Sum' THEN avg_annual_value END) AS avg_precipitation
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.4_precip_temp_nasa_1984_2022`
    WHERE year BETWEEN 1984 AND 2022
    GROUP BY year
),
fao_data AS (
    -- Filtra os dados da tabela FAO para obter os rendimentos de trigo (yield) entre 1984 e 2022
    SELECT
        year,
        value AS wheat_yield
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.fao_table`
    WHERE element = 'yield'
    AND year BETWEEN 1984 AND 2022
)
-- Junta os dados meteorológicos com os dados de rendimento de trigo usando o campo 'year'
SELECT
    n.year,
    n.avg_temperature,
    n.avg_precipitation,
    f.wheat_yield
FROM nasa_data n
JOIN fao_data f
ON n.year = f.year
ORDER BY n.year;
```




 

---


