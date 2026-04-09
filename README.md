# CrimeMYexplorer

Live site: <https://nqmn.github.io/crimeMYexplorer/>

`crimeMYexplorer` is a single-page map in `index.html` that visualizes Malaysian crime data by state or district and enriches it with population, density, income, poverty, unemployment, and inequality context.

The page uses Leaflet for the map, Chart.js for the yearly trend chart, Papa Parse for CSV export, and Turf.js to calculate area-based density values.

## What `index.html` does

The app starts with Malaysia administrative boundaries and a basemap, then fetches supporting reference data in the background:

- Crime data from `crime_district`
- Population from `population_district`
- Median household income from `hh_income_district` and `hh_income_state`
- Poverty from `hh_poverty_district` and `hh_poverty_state`
- Inequality from `hh_inequality_district` and `hh_inequality_state`
- Unemployment from `lfs_district` and `lfs_state`

When the user clicks **Fetch OpenDOSM Data**, the app first tries to load the official OpenDOSM crime dataset from the DOSM data file endpoint and then falls back to the `data.gov.my` API if needed. After loading, it populates the filters, recolors the map, shows a yearly trend line, and enables CSV download for a consolidated dataset.

## How to use the page

1. Open the live site.
2. Click **Fetch OpenDOSM Data**.
3. Choose the **View Level**.
4. Adjust the **Year** slider.
5. Narrow the results with **Crime Category** and **Crime Type**.
6. Hover on a state or district to inspect the tooltip.
7. Use the save button to download the consolidated CSV.

`States` aggregates crimes by state, while `Districts` aggregates crimes by district. `All Years` sums crime counts across all available years, while a specific year filters crime counts to that year only.

## What the map shows

The choropleth color is based on the total filtered crime count for each area.

- Darker red means a higher crime count relative to the current filtered view.
- Pale areas mean fewer or zero crimes in the current filtered view.

The tooltip can show:

- Total crimes
- Population
- Population density
- Crime rate per 100,000 residents
- Median income
- Relative poverty
- Unemployment
- Income Inequality (Gini coefficient)
- Socioeconomic risk score
- Crime-type breakdown

## Score calculation

The socioeconomic score shown in the tooltip and CSV is a weighted score from `0` to `10`.

Higher scores mean a location has weaker socioeconomic conditions according to the app's selected indicators. It is not a prediction of crime and it is not a measure of individual safety.

### Formula

The score is calculated as:

```text
Score =
  (IncomeRisk * 0.30) +
  (PovertyRisk * 0.30) +
  (UnemploymentRisk * 0.25) +
  (GiniRisk * 0.15)
```

Each component is first normalized to a `0` to `10` risk scale:

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

### Weighting

- Income: `30%`
- Relative poverty: `30%`
- Unemployment: `25%`
- Income Inequality (Gini coefficient): `15%`

### Missing-data behavior

- A score is only computed when both income and poverty data are available.
- If unemployment is missing, the app uses a fallback of `3.5`.
- If Income Inequality (Gini coefficient) is missing, the app uses a fallback of `0.35`.
- For socioeconomic series, the tooltip tries to use the selected year. If that year is unavailable, it falls back to the closest available earlier year, or the latest available record.

## How to interpret the score

The UI colors the score like this:

- `0.0` to `< 4.0`: lower socioeconomic risk
- `4.0` to `< 7.0`: moderate socioeconomic risk
- `7.0` to `10.0`: higher socioeconomic risk

Interpretation guidance:

- A higher score means lower income and/or higher poverty, unemployment, and inequality relative to the app's built-in normalization ranges.
- The score is best used for comparing areas within this app, not as an absolute national ranking.
- The score should be read together with the actual crime count and crime rate per 100,000, because a high crime count can simply reflect a larger population.
- `All Years` affects crime totals, but the socioeconomic indicators may come from the nearest available supporting year rather than a full multi-year average.

## CSV export

The download button exports a consolidated CSV with:

- State and district
- Category and type
- Year
- Crimes
- Population
- Area in square kilometers
- Population density
- Crime rate per 100,000
- Median income
- Relative poverty
- Unemployment
- Income Inequality (Gini coefficient)
- Socioeconomic risk score

## Data source

- OpenDOSM dataset page: <https://open.dosm.gov.my/data-catalogue/>


## Notes and limitations

- The map depends on external API and GeoJSON endpoints, so loading requires network access.
- District matching relies on normalized names, so some granular records may be grouped to standard map boundaries.
- The choropleth is based on filtered crime totals, not directly on the socioeconomic score.
- The score is a heuristic summary built from public indicators, not a causal model.
