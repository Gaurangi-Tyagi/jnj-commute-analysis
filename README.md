# Deutschlandticket Commute Analysis — J&J Medical GmbH, Norderstedt

## Overview

This project estimates how attractive public transport would be for employees commuting to Johnson & Johnson Medical GmbH (Robert-Koch-Straße 1, 22851 Norderstedt), and models the likely adoption potential of the Deutschlandticket across the workforce.

Since real employee addresses cannot be used, a synthetic employee dataset is generated to represent a realistic population living across Hamburg and the surrounding metro area. The analysis calculates door-to-door commute times using public transport, groups employees by commute time, scores each employee's likelihood of adopting the Deutschlandticket, and summarizes the findings both numerically and visually.

## Objective

Given a fixed workplace location, the analysis answers:

- How long would each employee's commute take door-to-door via public transport?
- What share of employees fall within 30, 45, 60, and over 60 minutes?
- How likely is each employee to adopt the Deutschlandticket, based on their commute profile?
- Which areas of Hamburg have strong public transport connectivity to Norderstedt, and which do not?
- What factors most influence adoption likelihood?

## Repository Contents

| File | Description |
|---|---|
| `commute_analysis.ipynb` | Main analysis notebook containing all steps, code, and explanations |
| `commute_map.html` | Standalone interactive map (workplace, stations, employees by adoption tier) |
| `README.md` | This file |

## Methodology

The notebook is organized into the following stages:

**1. Workplace geocoding**
The workplace address is geocoded programmatically using `geopy` and the Nominatim (OpenStreetMap) geocoding service.

**2. Synthetic employee data generation**
Employee home locations are randomly generated within a 25 km radius of central Hamburg. Points are distributed uniformly by area (not by radius) to avoid artificial clustering near the center, and longitude spacing is corrected for latitude to preserve realistic geographic distances.

**3. Public transport station reference data**
An attempt is made to retrieve live station data from the OpenStreetMap Overpass API. Due to reliability issues with the public Overpass endpoint (timeouts/rate limiting, documented in the notebook), the analysis falls back to a curated dataset of real HVV stations spanning the greater Hamburg network, compiled from public transit maps.

**4. Door-to-door commute time calculation**
For each employee, the analysis:
- Identifies the nearest station to their home using a geopandas spatial join (`sjoin_nearest`) on a projected coordinate system (EPSG:25832, UTM zone 32N) for accurate metric distances
- Models the home-to-station access leg using a distance-dependent mode assumption (walk under 1.5 km, bike 1.5–5 km, drive/park & ride beyond 5 km), reflecting standard transit accessibility catchment radii
- Calculates the transit leg using the straight-line distance between the employee's nearest station and the workplace's nearest station, inflated by a detour factor to approximate real route distances, with speed tiers that increase with distance to reflect faster regional/S-Bahn segments on longer trips
- Adds a fixed wait/transfer allowance
- Adds a last-mile leg from the workplace's nearest station to the office itself

**5. Commute-time grouping**
Employees are grouped into four bands: 30 minutes or less, 31–45 minutes, 46–60 minutes, and over 60 minutes.

**6. Deutschlandticket adoption scoring**
Each employee receives a rule-based, weighted adoption score (0–100), built from three explainable factors: commute time, home access mode, and station density near their home. Scores are grouped into High, Medium, and Low adoption potential tiers.

**7. Summary output and geographic analysis**
Results are aggregated by nearest home station to identify areas of strong and weak public transport connectivity relative to Norderstedt, and to surface the key factors driving adoption likelihood.

**8. Interactive map**
An interactive Folium map visualizes the workplace, all reference stations, and all synthetic employees, color-coded by adoption tier.

## Key Findings

- The majority of employees fall outside the 60-minute commute window, reflecting Norderstedt's position at the edge of the HVV network relative to a broadly distributed Hamburg workforce.
- Adoption potential skews toward the Low tier overall, driven primarily by commute time and the need to drive or bike to reach a station in many parts of the catchment area.
- Areas along the U1 corridor closest to Norderstedt (e.g. Ochsenzoll, Langenhorn Nord, Ohlsdorf) show the strongest connectivity and highest adoption scores.
- Areas such as Harburg, Bergedorf, and Blankenese show the weakest connectivity, with Harburg representing the largest single group of employees in a low-scoring area.
- The three factors most predictive of adoption likelihood are commute time, ease of reaching a station without a car, and the density of nearby stations.

Full figures, tables, and the underlying calculations are available in the notebook.

## Assumptions and Limitations

This analysis is built on documented, explainable assumptions rather than live routing data, and these should be read alongside the results:

- **Workplace coordinates** were geocoded via Nominatim; station coordinates were sourced from a manually curated reference list rather than a live transit API, due to reliability issues with the public OpenStreetMap Overpass endpoint during development.
- **Transit times** are estimated using distance-based speed tiers and a fixed detour factor rather than real HVV timetable data, since a live routing API was not available in this environment.
- **Adoption scoring weights** are analyst-defined assumptions, not calibrated against real survey or usage data.
- **Station coverage** reflects a representative selection of major rail and U-Bahn stations rather than an exhaustive bus stop inventory.

In a production setting, these components would be replaced with a live routing API (e.g. HVV's own journey planner API, or Google Maps Directions), and adoption weights would ideally be calibrated against actual employee survey or ticket usage data if available.

## Tools and Libraries

- Python 3.11
- pandas, numpy
- geopandas, shapely (spatial data handling and analysis)
- geopy (geocoding)
- folium (interactive mapping)
- requests (API calls)

## Running the Notebook

1. Create and activate the environment:
   ```
   conda create -n jnj-commute python=3.11 -y
   conda activate jnj-commute
   conda install -c conda-forge pandas numpy geopandas shapely folium jupyter ipykernel -y
   pip install geopy requests
   ```
2. Open `commute_analysis.ipynb` in Jupyter or VS Code and select the `jnj-commute` kernel.
3. Run all cells in order.
4. The interactive map is also available separately as `commute_map.html`.

## Author

Gaurangi Tyagi
