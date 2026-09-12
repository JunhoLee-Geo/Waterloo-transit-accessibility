# Waterloo Region Public Transit Accessibility Analysis

## Overview

This project analyzes public transit accessibility across Waterloo Region, Ontario by combining Grand River Transit (GRT) schedule and stop data with 2021 Census population data and municipal boundaries.

The analysis addresses two main questions:

1. How many residents live within 400 m or 800 m of existing GRT transit stops?
2. How does access to frequent weekday transit service differ across municipalities?

The project uses Python-based geospatial analysis to integrate GTFS transit data, Census dissemination areas, population data, and municipal boundaries.

---

## Key Findings

- Approximately **74.6%** of Waterloo Region residents are estimated to live within **400 m** of a GRT stop.
- Approximately **86.3%** are estimated to live within **800 m** of a GRT stop.
- Approximately **13.7%** are estimated to live more than **800 m** from a GRT stop.
- **52 dissemination areas (DAs)** were identified as priority underserved areas.
- These priority areas contain an estimated **49,712 residents** living beyond 800 m of transit.
- Only approximately **28.1%** of the regional population is estimated to live within 400 m of a stop classified as high-frequency under this analysis.

Among the three cities:

| Municipality | Within 400 m of Any Stop | Within 400 m of High-Frequency Stop |
|---|---:|---:|
| Cambridge | 83.8% | 14.8% |
| Kitchener | 82.9% | 35.9% |
| Waterloo | 81.0% | 42.9% |

These results demonstrate that **physical proximity to transit does not necessarily mean access to frequent transit service**.

---

## Visualizations

### Transit Proximity vs. High-Frequency Service

![Transit Proximity vs High-Frequency Service](figures/proximity_vs_frequency_access.png)

This comparison highlights the difference between having a transit stop nearby and having access to frequent weekday service.

### Municipality Transit Accessibility

![Municipality Transit Accessibility](figures/municipality_transit_accessibility.png)

Transit proximity is substantially higher in Cambridge, Kitchener, and Waterloo than in the surrounding townships.

### Priority Underserved Areas

![Priority Underserved Areas](figures/priority_underserved_areas.png)

Priority underserved dissemination areas were identified using both transit coverage and estimated population beyond 800 m.

### Top 10 Underserved Dissemination Areas

![Top 10 Underserved DAs](figures/top10_underserved_das.png)

This figure highlights the dissemination areas with the largest estimated populations living beyond 800 m of transit.

---

## Data Sources

### Grand River Transit (GRT)

GRT static GTFS data was used to obtain transit stop locations, routes, trips, stop times, and service calendar information.

Official source:

https://www.grt.ca/about-grt/open-data/

### Statistics Canada — 2021 Census

2021 Census dissemination area boundaries and population data were used to estimate the number of residents with access to transit.

Official dissemination area boundary files:

https://www150.statcan.gc.ca/n1/en/catalogue/92-169-X2021001

Boundary file reference guide:

https://www150.statcan.gc.ca/n1/pub/92-160-g/92-160-g2021001-eng.htm

The final study area contains **766 dissemination areas** with a total 2021 Census population of **587,165**.

### Region of Waterloo

Municipal boundary data was obtained from the Region of Waterloo GIS services and used to compare accessibility across:

- City of Cambridge
- City of Kitchener
- City of Waterloo
- Township of Wilmot
- Township of Woolwich
- Township of North Dumfries
- Township of Wellesley

GIS service used in this analysis:

https://gis.regionofwaterloo.ca/wamap/rest/services/HousingCatalogue/MapServer/17

---

## Methodology

### 1. Transit Stop Processing

GRT GTFS stop data was loaded and filtered to retain physical boarding locations.

Stop coordinates were converted into a GeoDataFrame and projected to:

**EPSG:26917 — NAD83 / UTM Zone 17N**

Using a projected coordinate reference system allowed distances and areas to be calculated in metres.

---

### 2. Transit Proximity Analysis

Two straight-line accessibility buffers were created around GRT stops:

- **400 m**
- **800 m**

Overlapping stop buffers were dissolved before calculating coverage so that overlapping service areas would not be double-counted.

The resulting transit coverage areas were intersected with 2021 Census dissemination areas.

---

### 3. Population Estimation

Population within transit coverage areas was estimated using area weighting.

```text
Estimated covered population
=
DA population × (intersection area / total DA area)
```

For example, if 60% of a dissemination area's land area falls within the 400 m transit buffer, approximately 60% of that DA's population is assumed to have 400 m transit access.

This provides an estimate rather than an exact count because population is not necessarily distributed evenly throughout each dissemination area.

---

### 4. Priority Underserved Areas

A dissemination area was classified as a **priority underserved area** when it met both conditions:

```text
Estimated 800 m transit coverage < 50%

AND

Estimated population beyond 800 m >= 500
```

Using this definition:

- **52 dissemination areas** were identified
- approximately **49,712 residents** in these areas were estimated to live beyond 800 m of transit

The analysis shows a particularly strong urban-rural accessibility gap across Waterloo Region.

---

## Service Frequency Analysis

Transit proximity alone does not describe the quality or intensity of transit service.

GTFS schedule data was therefore used to estimate weekday daytime service frequency.

### Representative Service Day

The analysis uses:

**Wednesday, September 9, 2026**

as a representative weekday from the available GTFS service calendar.

### Daytime Period

Daytime service was defined as:

```text
07:00 <= scheduled stop time < 19:00
```

This represents a 12-hour daytime service window.

For each stop, scheduled stop events during this period were counted and converted to:

```text
scheduled stop events per hour
```

---

## Service Frequency Classification

Stops were classified using the following analysis-specific thresholds:

```text
Low:       < 2 scheduled stop events/hour

Moderate:  2 to 4 scheduled stop events/hour

High:      > 4 scheduled stop events/hour
```

These categories were created specifically for this project and **are not official GRT service classifications**.

Of the stops with scheduled daytime service:

- approximately **17.0%** were classified as Low
- approximately **65.2%** were classified as Moderate
- approximately **17.8%** were classified as High

---

## High-Frequency Transit Accessibility

A second 400 m accessibility analysis was performed using only stops classified as high-frequency.

The result was compared with access to any GRT stop.

### Region-Wide Results

```text
Population within 400 m of any GRT stop:
74.6%

Population within 400 m of a high-frequency stop:
28.1%
```

This demonstrates an important distinction:

> A resident may live close to a transit stop without having access to frequent transit service.

---

## Municipality Comparison

| Municipality | Any Stop 400 m | High-Frequency 400 m |
|---|---:|---:|
| Cambridge | 83.8% | 14.8% |
| Kitchener | 82.9% | 35.9% |
| Waterloo | 81.0% | 42.9% |
| Wilmot | 25.3% | 0.0% |
| Woolwich | 18.9% | 0.1% |
| North Dumfries | 0.1% | 0.0% |
| Wellesley | 0.0% | 0.0% |

Cambridge has the highest estimated proximity to any transit stop among the three cities, but substantially lower high-frequency access than Waterloo or Kitchener.

Waterloo has the highest estimated high-frequency accessibility of the three cities despite having slightly lower overall stop proximity.

---

## Project Structure

```text
Waterloo-transit-accessibility/
│
├── data/
│   ├── processed/
│   │   ├── municipality_accessibility.csv
│   │   ├── waterloo_accessibility.gpkg
│   │   └── waterloo_da_population.gpkg
│   │
│   └── raw/
│       └── excluded from Git
│
├── figures/
│   ├── municipality_transit_accessibility.png
│   ├── priority_underserved_areas.png
│   ├── proximity_vs_frequency_access.png
│   └── top10_underserved_das.png
│
├── notebooks/
│   ├── 00_environment_test.ipynb
│   ├── 01_gtfs_exploration.ipynb
│   ├── 02_boundary_exploration.ipynb
│   ├── 03_census_exploration.ipynb
│   ├── 04_transit_accessibility.ipynb
│   ├── 05_visualization.ipynb
│   └── 06_service_frequency.ipynb
│
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Technologies Used

- Python
- pandas
- GeoPandas
- NumPy
- Matplotlib
- Shapely
- PyProj
- Jupyter Notebook
- GTFS
- GIS spatial analysis
- Spatial joins
- Buffer analysis
- Polygon intersections
- Area-weighted population estimation

---

## Reproducibility

### 1. Clone the repository

```bash
git clone https://github.com/j200292918/Waterloo-transit-accessibility.git
cd Waterloo-transit-accessibility
```

### 2. Create a virtual environment

```bash
python -m venv .venv
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Obtain the raw datasets

Raw datasets are excluded from this repository.

Download the required data from the official sources listed in the **Data Sources** section and place them in the appropriate directories under:

```text
data/raw/
```

### 5. Run the notebooks

Run the notebooks in the following order:

```text
00_environment_test.ipynb
01_gtfs_exploration.ipynb
02_boundary_exploration.ipynb
03_census_exploration.ipynb
04_transit_accessibility.ipynb
05_visualization.ipynb
06_service_frequency.ipynb
```

---

## Limitations

This analysis has several important limitations.

**Straight-line distance:**  
The 400 m and 800 m accessibility areas are Euclidean buffers. They do not account for the actual pedestrian network, sidewalks, crossings, highways, rivers, fences, or other physical barriers.

**Area-weighted population estimation:**  
Population is assumed to be uniformly distributed within each dissemination area. Actual residential locations may therefore differ from the estimates.

**Scheduled rather than observed service:**  
Frequency calculations use scheduled GTFS stop events rather than observed vehicle arrivals, delays, cancellations, or reliability.

**Stop events are not destination-specific frequency:**  
Scheduled stop events may represent multiple routes and directions. They should not be interpreted as the effective headway to a specific destination.

**Analysis-specific frequency threshold:**  
The definition of high-frequency service as more than four scheduled stop events per hour was created for this analysis and is not an official GRT classification.

**Different reference years:**  
Population distribution is based on the **2021 Census**, while transit service reflects the **September 2026 GTFS schedule**. The project therefore compares the latest complete small-area Census baseline used in the analysis with a more recent transit network.

---

## Future Improvements

Potential extensions of this analysis include:

- pedestrian street-network accessibility instead of straight-line buffers
- travel-time accessibility analysis
- access to employment, education, and essential services
- peak vs. off-peak service comparisons
- weekend service analysis
- population-weighted service frequency measures
- comparison across multiple GTFS service periods

---

## Author

**Junho Lee**  
University of Waterloo