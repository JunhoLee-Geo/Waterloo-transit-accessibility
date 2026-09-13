# Waterloo Region Public Transit Accessibility Analysis

## Overview

This project analyzes public transit accessibility across Waterloo Region using Grand River Transit (GRT) data and 2021 Census population data.

I used Python and GeoPandas to estimate how many residents live within 400 m and 800 m of GRT stops and to identify areas with limited transit coverage. I also used GTFS schedule data to compare access to nearby transit with access to frequent weekday service.

The analysis combines transit stops and schedules, Census dissemination areas, population data, and municipal boundaries.

## Key Results

About **74.6%** of Waterloo Region's population is estimated to live within **400 m** of a GRT stop, and **86.3%** within **800 m**. This leaves about **13.7%** of the population more than 800 m from a stop.

Using both transit coverage and population, I identified **52 priority underserved dissemination areas (DAs)**. These areas contain an estimated **49,712 residents** living more than 800 m from transit.

The frequency analysis showed a larger difference. While 74.6% of residents are estimated to live within 400 m of any GRT stop, only about **28.1%** live within 400 m of a stop classified as high-frequency in this project.

Among the three cities:

| Municipality | Any Stop within 400 m | High-Frequency Stop within 400 m |
|---|---:|---:|
| Cambridge | 83.8% | 14.8% |
| Kitchener | 82.9% | 35.9% |
| Waterloo | 81.0% | 42.9% |

Cambridge has the highest overall stop coverage of the three cities, but much lower high-frequency coverage than Kitchener and Waterloo. Waterloo has slightly lower overall stop coverage, but the highest high-frequency coverage of the three.

## Visualizations

### Transit Proximity vs. High-Frequency Service

![Transit Proximity vs High-Frequency Service](figures/proximity_vs_frequency_access.png)

This figure compares the share of the population within 400 m of any GRT stop with the share within 400 m of a high-frequency stop.

### Municipality Transit Accessibility

![Municipality Transit Accessibility](figures/municipality_transit_accessibility.png)

Transit coverage is much higher in Cambridge, Kitchener, and Waterloo than in the surrounding townships.

### Priority Underserved Areas

![Priority Underserved Areas](figures/priority_underserved_areas.png)

Priority underserved DAs were identified using both 800 m transit coverage and the estimated population living beyond 800 m.

### Top 10 Underserved Dissemination Areas

![Top 10 Underserved DAs](figures/top10_underserved_das.png)

This figure shows the ten DAs with the largest estimated populations living more than 800 m from a GRT stop. The labels are Statistics Canada dissemination area identifiers (DAUIDs).

## Data Sources

### Grand River Transit (GRT)

I used GRT's static GTFS data for stop locations, routes, trips, scheduled stop times, and service calendar information.

Official source:

https://www.grt.ca/about-grt/open-data/

### Statistics Canada — 2021 Census

I used 2021 Census dissemination area boundaries and population data to estimate the number of residents within the transit coverage areas.

Dissemination area boundary files:

https://www150.statcan.gc.ca/n1/en/catalogue/92-169-X2021001

Boundary file reference guide:

https://www150.statcan.gc.ca/n1/pub/92-160-g/92-160-g2021001-eng.htm

The final study area contains **766 dissemination areas** with a total 2021 Census population of **587,165**.

### Region of Waterloo

Municipal boundary data was used to compare transit accessibility across:

- Cambridge
- Kitchener
- Waterloo
- Wilmot
- Woolwich
- North Dumfries
- Wellesley

GIS service:

https://gis.regionofwaterloo.ca/wamap/rest/services/HousingCatalogue/MapServer/17

## Method

### Transit Stop Processing

I loaded the GRT GTFS stop data and kept the physical boarding locations needed for the analysis.

The stop coordinates were converted to a GeoDataFrame and projected to:

**EPSG:26917 — NAD83 / UTM Zone 17N**

I used a projected coordinate system so that distance and area calculations could be done in metres.

### Transit Coverage

I created **400 m** and **800 m** straight-line buffers around GRT stops.

Overlapping buffers were dissolved before calculating coverage so the same area would not be counted more than once. I then intersected the coverage areas with the Census dissemination areas.

### Population Estimation

Census population is reported for an entire DA rather than for individual locations within it, so I used area weighting to estimate the population inside each transit coverage area.

```text
Estimated covered population =
DA population × (intersection area / total DA area)
```

For example, if 60% of a DA's area falls within the 400 m buffer, the analysis estimates that 60% of its population has transit access within 400 m.

This is an estimate because population is not necessarily distributed evenly within each DA.

### Priority Underserved Areas

For this project, I classified a DA as priority underserved when it met both of the following conditions:

```text
Estimated 800 m transit coverage < 50%

AND

Estimated population beyond 800 m >= 500
```

Using these criteria, **52 DAs** were identified. Together, they contain an estimated **49,712 residents** living more than 800 m from a GRT stop.

## Service Frequency

Distance to a stop does not show how often transit is available, so I also used the GTFS schedule to look at weekday daytime service frequency.

I used **Wednesday, September 9, 2026** as a representative weekday from the available GTFS service calendar.

The daytime period was defined as:

```text
07:00 <= scheduled stop time < 19:00
```

For each stop, I counted scheduled stop events during this 12-hour period and calculated the average number of scheduled stop events per hour.

For this project, stops were grouped into three categories:

```text
Low:       < 2 scheduled stop events/hour
Moderate:  2–4 scheduled stop events/hour
High:      > 4 scheduled stop events/hour
```

These thresholds were created for this analysis and are **not official GRT service classifications**.

Among stops with scheduled daytime service:

- 17.0% were classified as Low
- 65.2% were classified as Moderate
- 17.8% were classified as High

I then repeated the 400 m accessibility analysis using only the high-frequency stops.

Region-wide:

```text
Population within 400 m of any GRT stop:           74.6%
Population within 400 m of a high-frequency stop:  28.1%
```

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

The three cities have much higher stop coverage than the surrounding townships.

There are also noticeable differences within the urban areas. Cambridge has the highest access to any stop among the three cities but much lower high-frequency access. Waterloo has the highest high-frequency accessibility at 42.9%.

## What I Would Look At Next

The results point to two different transit issues: areas where stops are not nearby and areas where stops are available but service is less frequent.

The **52 priority underserved DAs** could be a starting point for looking at where route extensions, new stops, or other service changes may be worth studying. Areas that already have good stop coverage but lower-frequency service could instead be examined for possible service frequency improvements.

This analysis alone is not enough to recommend exact stop locations or route changes. A more detailed study could include the pedestrian street network, ridership, travel times, destinations, and the existing route network.

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

## Tools

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
- spatial joins
- buffer analysis
- polygon intersections
- area-weighted population estimation

## Reproducing the Analysis

Clone the repository:

```bash
git clone https://github.com/JunhoLee-Geo/Waterloo-transit-accessibility.git
cd Waterloo-transit-accessibility
```

Create a virtual environment:

```bash
python -m venv .venv
```

Install the required packages:

```bash
pip install -r requirements.txt
```

Raw datasets are not included in the repository. Download them from the sources listed above and place them under:

```text
data/raw/
```

Then run the notebooks in order:

```text
00_environment_test.ipynb
01_gtfs_exploration.ipynb
02_boundary_exploration.ipynb
03_census_exploration.ipynb
04_transit_accessibility.ipynb
05_visualization.ipynb
06_service_frequency.ipynb
```

## Limitations

**Straight-line distance:** The 400 m and 800 m buffers do not follow the actual pedestrian network. Roads, highways, rivers, crossings, fences, and other barriers can affect real walking distance.

**Population estimation:** Area weighting assumes that population is evenly distributed within each DA. Actual residential locations may differ from these estimates.

**Scheduled service:** The frequency analysis uses scheduled GTFS data, so delays, cancellations, and actual service reliability are not included.

**Stop events:** Scheduled stop events can include different routes and directions, so they should not be interpreted as the actual waiting time for a specific destination.

**Frequency thresholds:** The Low, Moderate, and High categories were created for this project and are not official GRT classifications.

**Different reference years:** Population data comes from the 2021 Census, while transit service is based on the September 2026 GTFS schedule.

## Possible Improvements

Possible next steps include:

- pedestrian network distance instead of straight-line buffers
- travel-time accessibility
- access to jobs, schools, and essential services
- peak and off-peak service comparisons
- weekend service analysis
- population-weighted service frequency
- comparisons between different GTFS service periods

## Author

**Junho Lee**  
University of Waterloo