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
    -- Calculates the mean and standard deviation of annual_temp for each variable
    SELECT
        variable,
        AVG(annual_temp) AS mean_annual_temp,
        STDDEV(annual_temp) AS stddev_annual_temp
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table`
    GROUP BY variable
),
annual_temps_with_zscore AS (
    -- Joins the statistics with the original data and calculates the Z-score
    SELECT
        nt.year,
        nt.variable,
        nt.annual_temp,
        s.mean_annual_temp,
        s.stddev_annual_temp,
        (nt.annual_temp - s.mean_annual_temp) / s.stddev_annual_temp AS zscore
    FROM `my-portifolio-434417.Netherlands_Agricultural_and_Meteorological_Data.nasa_table` nt
    JOIN stats s
    ON nt.variable = s.variable
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



 

---


