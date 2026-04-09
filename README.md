# CrimeMYexplorer

Live site: <https://nqmn.github.io/crimeMYexplorer/>

`crimeMYexplorer` is a single-page Malaysia crime explorer in `index.html`. It combines live district-level crime data with administrative boundaries and socioeconomic indicators so users can switch between a crime heatmap and a socioeconomic risk map.

The page uses Leaflet for mapping, Chart.js for the yearly trend chart, Papa Parse for CSV export, and Turf.js for area and density calculations.

## What `index.html` does

On load, the app:

- Initializes a Leaflet map centered on Malaysia
- Loads Malaysia state and district GeoJSON boundaries from GitHub
- Pre-calculates polygon area for density metrics
- Fetches supporting reference series in the background from `data.gov.my`

Background reference datasets:

- `population_district`
- `hh_income_district`
- `hh_income_state`
- `hh_poverty_district`
- `hh_poverty_state`
- `hh_inequality_district`
- `hh_inequality_state`
- `lfs_district`
- `lfs_state`

When the user clicks **Fetch Live API Data**, the app loads the `crime_district` dataset from `https://api.data.gov.my/data-catalogue/?id=crime_district`, enables the controls, updates the map, shows the yearly trend chart, and reveals a rotating fact banner based on a random crime record.

## How to use the page

1. Open the live site.
2. Click **Fetch Live API Data**.
3. Choose a **Map Layer**:
   `Crime Map` or `Risk Map`.
4. Choose a **View Level**:
   `States` or `Districts`.
5. Adjust the **Year** slider.
6. Filter by **Crime Category** and **Crime Type**.
7. Hover an area to inspect the tooltip.
8. Click the save icon to export the consolidated CSV.
9. Click the **Did you know?** banner to refresh the fact.

`All Years` sums crimes across all available years. A specific year filters crime counts to that year only.

## Map layers

### Crime Map

The default choropleth colors areas by filtered total crime count.

- Darker red means more filtered crimes
- Pale areas mean fewer or zero filtered crimes

The legend is scaled relative to the current filtered maximum.

### Risk Map

The alternate choropleth colors areas by a `0` to `10` socioeconomic risk score.

- `8` to `10`: Very High
- `6` to `< 8`: High
- `4` to `< 6`: Medium
- `2` to `< 4`: Low-Medium
- `0` to `< 2`: Low
- No fill emphasis: no supporting data available

This score is a contextual socioeconomic summary. It is not a crime prediction and not a measure of personal safety.

## Tooltip contents

Depending on the selected area and available data, the tooltip can show:

- Total crimes
- Population
- Population density per square kilometer
- Crime rate per 100,000 residents
- Median income
- Relative poverty
- Unemployment
- Income inequality (Gini coefficient)
- Socioeconomic risk score
- Crime-type breakdown

For supporting metrics, the app first tries the selected year. If that year is unavailable, it falls back to the closest available earlier year, or the latest available record.

## Score calculation

The socioeconomic score is calculated on a `0` to `10` scale:

```text
Score =
  (IncomeRisk * 0.30) +
  (PovertyRisk * 0.30) +
  (UnemploymentRisk * 0.25) +
  (GiniRisk * 0.15)
```

Each component is normalized first:

```text
IncomeRisk =
  (1 - ((clamp(income, 2500, 12500) - 2500) / 10000)) * 10

PovertyRisk =
  (clamp(relative_poverty, 0, 25) / 25) * 10

UnemploymentRisk =
  ((clamp(unemployment, 2, 10) - 2) / 8) * 10

GiniRisk =
  ((clamp(gini, 0.25, 0.45) - 0.25) / 0.20) * 10
```

Weights:

- Income: `30%`
- Relative poverty: `30%`
- Unemployment: `25%`
- Gini coefficient: `15%`

Missing-data behavior:

- A score is only computed when both income and poverty are available
- If unemployment is missing, the app uses `3.5`
- If Gini is missing, the app uses `0.35`
- In the UI, `All Years` uses 2022 as the default target year for income, poverty, and inequality, and 2023 for unemployment before fallback logic is applied

## Yearly trend chart

After crime data loads, the app displays a line chart of filtered reported crimes by year.

- The chart excludes national rollup rows and `All` district/type aggregates
- Changing category or type updates the series
- The currently selected year is highlighted on the chart

## CSV export

The save button exports `malaysia_crime_socioeconomic_data.csv` built from the loaded crime rows and the closest matching supporting indicators.

Export columns:

- `State`
- `District`
- `Category`
- `Type`
- `Year`
- `Crimes`
- `Population`
- `Area_SqKm`
- `Density_Per_SqKm`
- `Crime_Rate_Per_100k`
- `Median_Income_RM`
- `Relative_Poverty_Pct`
- `Unemployment_Pct`
- `Gini_Coefficient`
- `Socioeconomic_Risk_Score`

## Data sources

- Crime and socioeconomic datasets: <https://api.data.gov.my/data-catalogue/>
- OpenDOSM dataset page: <https://open.dosm.gov.my/data-catalogue/>
- State boundaries: <https://github.com/nullifye/malaysia.geojson>
- District boundaries: <https://github.com/nullifye/malaysia.geojson>
- Basemap tiles: CARTO Voyager / OpenStreetMap contributors

## Notes and limitations

- The app requires network access because it loads live API and GeoJSON resources at runtime.
- District and state matching relies on normalized names, so some records may be grouped to the nearest supported boundary name.
- Live API responses may be incomplete or may not fully load in some sessions because of upstream API limitations.
- The crime layer is based on filtered crime totals, not rates.
- The risk layer is a heuristic built from public socioeconomic indicators, not a causal or predictive model.
- Population values are normalized in code when the source appears to be reported in thousands.

## Support

If you are interested in collaborating to improve this heatmap for public-benefit use cases, please get in touch.
