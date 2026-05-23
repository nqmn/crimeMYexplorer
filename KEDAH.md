# Kedah Crime Explorer (`kedah.html`)

`kedah.html` is a Kedah-focused derivative of the national `index.html` crime explorer. It isolates all crime, boundary, and socioeconomic data to the state of Kedah only, with the map pre-centered and zoomed into Kedah at district-level granularity.

## What `kedah.html` does

On load, the app:

- Initializes a Leaflet map centered on **Kedah** (`[6.12, 100.5]`, zoom `9`)
- Loads Malaysia state and district GeoJSON boundaries, then **filters to Kedah only** (`state === 'KDH'`)
- Pre-calculates polygon area for Kedah's 12 districts
- Auto-loads **78 police stations** from inline GeoJSON data and renders markers on the map
- Fetches supporting reference datasets from `data.gov.my`, **filtered to Kedah rows only** via `isKedahRow()`
- Applies **pastel district colors** to each of Kedah's 12 districts for visual distinction

When the user clicks **Fetch Live API Data**, the app:

- Loads the `crime_district` dataset
- **Filters `rawData` to `state === 'kedah'` only**
- Enables controls, updates the district-level map, shows the yearly trend chart, and reveals a rotating fact banner

### Kedah-specific modifications from `index.html`

| Aspect | `index.html` | `kedah.html` |
|---|---|---|
| Map center | `[4.2105, 101.9758]` (Malaysia) | `[6.12, 100.5]` (Kedah) |
| Default zoom | `6` | `9` |
| Default view level | `state` | `district` |
| GeoJSON features | All Malaysia | Kedah only (`KDH`) |
| Crime data | All states | Kedah only |
| Socioeconomic data | All states | Kedah only (`isKedahRow`) |
| Page title | Malaysia Crime Data Visualization | Kedah Crime Data Visualization |
| Header label | Crime Map MY | Crime Map Kedah |
| Police stations | Not included | 78 stations, auto-loaded (inline GeoJSON) |
| District colors | White (`#f8fafc`) | Pastel per-district colors |
| Note block | Present | Removed |

### Kedah districts covered

1. Baling
2. Pendang
3. Padang Terap
4. Kuala Muda
5. Bandar Baharu
6. Yan
7. Kota Setar
8. Kulim
9. Kubang Pasu
10. Sik
11. Pokok Sena
12. Langkawi

## Control panel layout

1. **Header** — Crime Map Kedah branding + CSV download button
2. **Amenities** — Police station checkboxes (HQ, Balai Polis, Pondok Polis, Trafik, Specialist, Komuniti) + coverage radius slider (0–20 km, default 0 km)
3. **Fetch Live API Data** button — loads crime dataset above Map Filters
4. **Map Filters** (collapsible, default collapsed) — Crime/Risk layer, Year slider, Category, Type
5. **Yearly Trend** — line chart of filtered reported crimes by year
6. **Did You Know?** — floating panel at bottom-left, click to refresh fact

## How to use the page

1. Open `kedah.html`.
2. Police station markers load automatically. Toggle station types via checkboxes.
3. Adjust **Coverage Radius** slider to show patrol/response circles around stations.
4. Click **Fetch Live API Data** to load crime data.
5. Expand **Map Filters** to choose Crime/Risk layer, Year, Category, Type.
6. Hover a district to inspect the tooltip.
7. Click a district to open the detail panel on the right.
8. Click the save icon to export the Kedah-only consolidated CSV.
9. Click the **Did you know?** banner at bottom-left to refresh the fact.

## Police station amenities

### Auto-load behavior

Police stations load automatically on page init — no button click needed. The GeoJSON data is embedded inline in `kedah.html` (no external fetch, avoids CORS issues with `file://` protocol).

### Station type checkboxes

| Checkbox | Type | Icon | Color | Default |
|---|---|---|---|---|
| HQ | headquarters | HQ | `#1e3a8a` (dark blue) | Checked |
| Balai Polis | balai | BP | `#dc2626` (red) | Checked |
| Pondok Polis | pondok | PP | `#f59e0b` (amber) | Unchecked |
| Trafik | traffic | TP | `#2563eb` (blue) | Checked |
| Specialist | specialist | SP | `#7c3aed` (purple) | Checked |
| Komuniti | komuniti | KP | `#059669` (green) | Checked |

Pondok Polis is unchecked by default to reduce visual clutter (11 pondok stations hidden by default, 67 shown).

### Coverage radius slider

- Range: 0–20 km
- Default: 0 km (no circles shown)
- Circles use dashed border with 8% fill opacity, matching station type color
- Useful for identifying coverage gaps between stations

### Data fetching methodology

Balai polis locations are sourced from **OpenStreetMap** via the Overpass API:

1. **Query**: Overpass API queried for all OSM elements tagged `amenity=police` within the Kedah state boundary (`area["name"="Kedah"]`).
2. **Endpoint**: `https://lz4.overpass-api.de/api/interpreter` (public mirror with higher throughput).
3. **Query syntax**:
   ```text
   [out:json][timeout:25];
   area["name"="Kedah"]->.kedah;
   (node["amenity"="police"](area.kedah);
    way["amenity"="police"](area.kedah);)->.out;
   out tags center;
   ```
4. **Initial result**: 62 features from OSM (nodes with exact coordinates + ways resolved to center points).
5. **Manual additions**: 16 stations added manually for districts with incomplete OSM coverage (notably Langkawi: 1?8, missing IPDs, missing BP Sungai Petani, etc.).
6. **Classification**: Each station is categorized by its name:
   - `headquarters` — Ibu Pejabat, Kontinjen, Batalion
   - `balai` — Balai Polis (full station)
   - `pondok` — Pondok Polis (mini station)
   - `traffic` — Trafik / Traffic Police
   - `specialist` — Bahagian Siasatan, Pencegahan Jenayah, etc.
   - `komuniti` — Balai Polis Komuniti
7. **District assignment**: Each station is assigned to one of Kedah's 12 districts by nearest-centroid distance matching against district center coordinates.
8. **Output files**: `kedah_police_stations.geojson` (standalone) + inline `POLICE_STATIONS_GEOJSON` constant in `kedah.html`.
9. **Sync**: Both files contain the same 78 features. The inline version is the authoritative source for the app (avoids CORS). The standalone file is kept for reuse.

#### Station summary by district

| District | Balai | Pondok | HQ | Traffic | Specialist | Komuniti | Total |
|---|---|---|---|---|---|---|---|
| Kota Setar | 6 | 1 | 2 | 2 | 1 | 0 | 12 |
| Kuala Muda | 7 | 3 | 1 | 1 | 0 | 1 | 13 |
| Kubang Pasu | 3 | 2 | 1 | 0 | 0 | 1 | 7 |
| Pendang | 3 | 3 | 1 | 1 | 0 | 0 | 8 |
| Kulim | 3 | 0 | 2 | 1 | 0 | 0 | 6 |
| Langkawi | 4 | 3 | 1 | 0 | 0 | 0 | 8 |
| Yan | 3 | 2 | 0 | 0 | 0 | 0 | 5 |
| Baling | 2 | 0 | 1 | 0 | 0 | 0 | 3 |
| Bandar Baharu | 2 | 1 | 1 | 0 | 0 | 0 | 4 |
| Sik | 1 | 2 | 1 | 0 | 0 | 0 | 4 |
| Padang Terap | 2 | 0 | 1 | 0 | 0 | 0 | 3 |
| Pokok Sena | 2 | 0 | 0 | 0 | 0 | 0 | 2 |
| **Total** | **38** | **11** | **10** | **5** | **1** | **2** | **78** |

*Note: "Specialist" count moved from Traffic; "Komuniti" separated from Balai. Totals may differ from earlier versions due to reclassification and manual additions.*

#### Limitations of OSM data

- OSM is community-maintained; some balai polis may be missing, misplaced, or have outdated names
- Coordinates are approximate (especially for `way` features resolved to center points)
- Phone numbers and operating hours are often not tagged in OSM
- District assignment is based on nearest-centroid heuristic, not official PDRM boundaries
- Langkawi stations were largely added manually (OSM only had 1 entry for Langkawi)
- For production use, this dataset should be validated against official PDRM records

## District colors

Each district has a unique pastel fill color for visual distinction when no crime data is overlaid:

| District | Color | Hex |
|---|---|---|
| Kota Setar | Soft red | `#fde8e8` |
| Kubang Pasu | Soft blue | `#e8f0fe` |
| Padang Terap | Soft orange | `#fef3e2` |
| Sik | Soft green | `#e8f5e9` |
| Baling | Soft pink | `#fce4ec` |
| Kulim | Soft indigo | `#e8eaf6` |
| Bandar Baharu | Soft amber | `#fff8e1` |
| Pendang | Soft purple | `#f3e5f5` |
| Yan | Soft teal | `#e0f2f1` |
| Kuala Muda | Soft rose | `#ffebee` |
| Pokok Sena | Soft light-blue | `#e1f5fe` |
| Langkawi | Soft peach | `#fff3e0` |

When crime data is loaded and a district has crime count > 0, the crime heatmap color overrides the district color. When crime count is 0, the district pastel shows at 50% opacity.

## Data sources

- Crime and socioeconomic datasets: <https://api.data.gov.my/data-catalogue/>
- OpenDOSM dataset page: <https://open.dosm.gov.my/data-catalogue/>
- Police station locations: <https://www.openstreetmap.org> via Overpass API (`amenity=police` in Kedah)
- Police station GeoJSON: `kedah_police_stations.geojson` (local file, 78 stations) + inline in `kedah.html`
- State boundaries: <https://github.com/nullifye/malaysia.geojson>
- District boundaries: <https://github.com/nullifye/malaysia.geojson>
- Basemap tiles: CARTO Voyager / OpenStreetMap contributors

---

## Future Use Cases

The following use cases are planned or proposed to evolve `kedah.html` from a retrospective analytics dashboard into an **operational tool** for law enforcement and public safety planning.

### 1. Balai Polis Layer — Police Station Locations ? IMPLEMENTED

This use case is now fully implemented. See the **Police station amenities** section above for details.

Remaining improvements:
- Validate coordinates against official PDRM records
- Add phone numbers and operating hours from official sources
- Real-time station status (open/closed/busy)

### 2. Daily Crime Feed — Real-Time Incident Updates

Replace the current annual/batch crime data with a **daily-updated crime incident feed**.

- **Data source**: PDRM internal feed (via secure API) or DOSM daily crime statistics release
- **Benefits**:
  - Shift from backward-looking analysis to **near-real-time situational awareness**
  - Enable same-day identification of emerging crime spikes
  - Provide empirical evidence for daily patrol briefings
- **Implementation notes**:
  - Auto-refresh every 15 minutes via `setInterval` or WebSocket
  - Date range picker replacing the current year-only slider
  - Badge count on the "Fetch" button showing number of new incidents since last load
  - Incident markers (clustered) with type, time, and status icons

### 3. Daily Heatmap Hotspot — Dynamic Crime Density

Upgrade the static choropleth to a **daily-updated kernel density heatmap** that highlights micro-level crime hotspots.

- **Data source**: Same daily crime feed, processed into point-level coordinates
- **Benefits**:
  - Show *where* crime is concentrating *today*, not just *how much* occurred last year
  - Reveal micro-hotspots that district-level aggregation hides (e.g., a single taman vs. an entire daerah)
  - Provide evidence-based justification for resource reallocation
- **Implementation notes**:
  - Leaflet.heat plugin for real-time heatmap rendering
  - Configurable time window: last 24h, 7 days, 30 days
  - Hotspot intensity legend with dynamic thresholds
  - Animated pulse effect on emerging hotspots (new in last 3 hours)

### 4. MPV Patrol Routing — Focused Deployment

Generate **suggested patrol routes** for Mobile Patrol Vehicle (MPV) units based on today's heatmap hotspots.

- **Data source**: Daily heatmap output + road network (OpenStreetMap) + balai polis locations
- **Benefits**:
  - Direct crime data ? patrol action pipeline
  - Reduce response time by pre-positioning MPVs near predicted hotspots
  - Equalize patrol coverage across shifts — no district left uncovered
  - Measure patrol effectiveness over time (crime reduction in patrolled zones)
- **Implementation notes**:
  - Shortest-path routing via OSRM or GraphHopper API
  - Each MPV assigned a priority zone based on hotspot severity
  - Route card UI: shift time, zone, estimated patrol duration, key risk points
  - Daily route summary exportable as PDF for shift handover

### 5. Crime Forecasting — Predictive Hotspot Modeling

Apply time-series and spatial models to **predict** where crime is likely to increase next.

- **Data source**: Historical crime data + socioeconomic indicators + seasonal patterns
- **Benefits**:
  - Proactive deployment instead of reactive patrolling
  - Identify seasonal patterns (e.g., break-ins rise before festive seasons)
  - Early warning system for districts showing upward trend signals
- **Implementation notes**:
  - Backend model (Prophet / ARIMA / spatial Poisson) served via lightweight API
  - Confidence intervals displayed as translucent overlay zones
  - Toggle between "Actual" and "Forecast" map modes
  - Alert notifications when a district's predicted count exceeds a configurable threshold

### 6. Public Safety Dashboard — Community Transparency

A public-facing view of the Kedah crime map that shows **anonymized, aggregated** data for community awareness.

- **Data source**: Same crime dataset, filtered to remove sensitive details
- **Benefits**:
  - Build public trust through transparency
  - Empower residents to make informed safety decisions (e.g., avoid poorly lit areas at night)
  - Encourage community policing — Rukun Tetangga zones can self-organize around known hotspots
- **Implementation notes**:
  - Separate `public.html` with reduced data granularity (no exact addresses, no victim info)
  - Safety tips overlay per district
  - "Report Suspicious Activity" form linking to PDRM portal
  - Multi-language support (BM, English, ??, ?????)

### 7. Emergency Response Integration — 999 Linkage

Connect the crime map to the **MERS 999** emergency response system for real-time incident dispatch visualization.

- **Data source**: MERS 999 dispatch feed (via secure integration)
- **Benefits**:
  - Show live incident locations as they are reported
  - Track MPV/ambulance response in real time on the map
  - Post-incident analysis: response time vs. distance from nearest balai polis
- **Implementation notes**:
  - Read-only view of dispatch events (no write access to 999 system)
  - Color-coded markers by incident type (red = violent, orange = property, blue = traffic)
  - Auto-zoom to active incident on selection
  - Response time benchmarking dashboard per district

### 8. Inter-Agency Coordination — Multi-Layer Governance

Enable **PDRM, APM, RELA, and JPBM** to share a common operational picture.

- **Data source**: Aggregated feeds from multiple agencies, with role-based access
- **Benefits**:
  - Eliminate information silos between enforcement agencies
  - Joint operations planning on a shared map
  - Unified after-action review with synchronized timeline playback
- **Implementation notes**:
  - Role-based layer visibility (PDRM sees crime, APM sees maritime, RELA sees crowd control zones)
  - Shared annotation/drawing tools on the map
  - Operation event log with timestamps and responsible agency
  - SSO integration with agency credentials

---

## Notes and limitations

- The app requires network access because it loads live API and GeoJSON resources at runtime.
- Police station data is embedded inline (no external fetch) to avoid CORS issues when opening `kedah.html` via `file://` protocol.
- District matching relies on normalized names, so some Kedah records may not match if the API uses variant spellings.
- Live API responses may be incomplete or may not fully load in some sessions because of upstream API limitations.
- The crime layer is based on filtered crime totals, not rates.
- The risk layer is a heuristic built from public socioeconomic indicators, not a causal or predictive model.
- Population values are normalized in code when the source appears to be reported in thousands.
- Currently, the data is updated annually. Daily and real-time features listed under Future Use Cases are aspirational and require data partnerships with PDRM and DOSM.
- Browser caching: use **Ctrl+Shift+R** (hard refresh) to ensure the latest version of `kedah.html` is loaded.

## Support

If you are interested in collaborating to improve this heatmap for public-benefit use cases — especially the MPV patrol routing, daily crime feed, or emergency response integration — please get in touch.