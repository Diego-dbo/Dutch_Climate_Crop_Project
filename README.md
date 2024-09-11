# Dutch Wheat and Climate Dynamics: A Data Analysis (1961-2022)

## **Key Insights**


## 1. Evolution of Wheat Production in the Netherlands
- **Yield**: Between 1961 and 2022, wheat yield in the Netherlands grew by **138.93%**, increasing from 3.93 to 9.39 tons per hectare. Annual yield averages provided by the FAO, covering the period from **1961 to 2022**, were used (Dashboard - Temporal Analysis of Wheat Yield).
  
- **Production**: During the same period, wheat production increased by **141.19%**, reflecting improvements in agricultural practices and positive influences of climate conditions. The annual production averages were also sourced from FAO data, covering **1961 to 2022** (Dashboard - Temporal Analysis of Wheat Production).
  
- **Harvested Area**: The harvested area remained stable, with a marginal increase of **0.78%**. This suggests that the production increase was primarily driven by higher yield per hectare rather than significant expansion of the cultivated area. Annual harvested area averages were obtained from FAO data, covering **1961 to 2022** (Dashboard - Temporal Analysis of Wheat Area).

---

## 2. Correlations Between Agricultural and Climatic Variables
- **Production vs. Yield**: The analysis revealed a strong positive correlation of **0.90** between production and yield, indicating that production increases are directly associated with higher productivity per hectare. This analysis covers data from **1961 to 2022** (Dashboard - Correlation Analysis: Yield vs Production).
  
- **Yield vs. Temperature**: A moderate correlation of **0.37** was identified between yield and annual average temperature. Higher temperatures are associated with increased yield, up to a certain limit. Annual temperature averages were derived from NASA data, covering **1984 to 2022** (Dashboard - Correlation Analysis: Yield vs Temperature).
  
- **Yield vs. Precipitation**: A weak negative correlation of **-0.19** was observed between yield and annual precipitation. This suggests that excessive precipitation may negatively impact yield, possibly due to excess moisture during cultivation or harvest. Annual precipitation averages were extracted from NASA data, covering **1984 to 2022** (Dashboard - Correlation Analysis: Yield vs Precipitation).

---

## 3. Temporal Analysis of Climatic Conditions
- **Temperature**: Annual average temperature increased by **26.37%** between 1984 and 2022, which may indicate climate changes that positively impact wheat growth. Annual temperature averages provided by NASA were used (Dashboard - Long-term Temperature Trends).
  
- **Precipitation**: During the same period, annual average precipitation showed a reduction of **8.83%**, suggesting changes in rainfall patterns that could affect agricultural practices, such as irrigation. Annual precipitation averages were also sourced from NASA data (Dashboard - Long-term Precipitation Trends).
  
- **Monthly Variations**: Winter months (December to February) showed the highest levels of precipitation, while April recorded the lowest levels. The highest temperatures occurred in the summer months (June to August), peaking in July and August. Monthly temperature and precipitation averages provided by NASA, covering **1984 to 2022**, were used (Dashboard - Monthly Weather Variables).

---

## 4. Climatic Outliers Analysis
- **Z-Score Analysis**: An outlier analysis was conducted using a Z-score with a threshold of 3, and no outliers were detected. This indicates that, between 1984 and 2022, climatic variations remained within a stable range, with no extreme climatic events classified as outliers (Dashboard - Outlier Detection).

---

## 5. Impact of Climatic Conditions on Wheat Production
- **Temperature and Yield**: The analysis showed a moderate correlation between temperature and wheat yield, suggesting that the gradual increase in temperatures has a moderately positive impact on yield.
  
- **Precipitation and Yield**: The weak negative correlation between precipitation and yield indicates that excessive precipitation has a limited but potentially harmful effect on yield. Wheat appears to be more sensitive to temperature variations than precipitation.

---

## Conclusion
- **Significant Increase in Production and Yield**: Between 1961 and 2022, wheat production and yield in the Netherlands increased significantly, driven primarily by higher productivity per hectare.
  
- **Moderate Influence of Temperature**: The gradual increase in temperature over time contributed moderately to the increase in wheat yield.
  
- **Stability in Climatic Conditions**: The outlier analysis did not detect any extreme climatic events between 1984 and 2022, indicating that climatic conditions remained within stable patterns, without major disruptions that would have drastically impacted production.

---

## Interactive Visualization
An **interactive dashboard** was created in Tableau to facilitate the visualization of the analysis results, allowing dynamic exploration of yield, production, harvested area, temperature, and precipitation variables over time.

Access the interactive dashboard through this link: [**Interactive Dashboard on Tableau**](https://public.tableau.com/app/profile/diego.oliveira5592/viz/DutchWheatandClimateDynamicsADataAnalysis1961-2022/Dashboard_IntegratedAnalysisofWheatProductionandClimateTrends1961-2022#1).

---

## Scripts
The SQL scripts used for the analysis are available in the **scripts** folder for reference and further use. These scripts were employed to clean, process, and analyze the FAO and NASA datasets.

---

This report presents a comprehensive analysis of the relationship between agricultural and climatic variables, using data from FAO (1961-2022) and NASA (1984-2022), highlighting key insights about the climate's impact on wheat production in the Netherlands.
