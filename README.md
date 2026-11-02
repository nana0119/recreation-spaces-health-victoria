# Recreational Spaces and Health Outcomes Across Victoria

## Overview

This project investigates whether access to recreational spaces helps explain differences in physical activity and health outcomes across Victoria's 79 Local Government Areas (LGAs), after accounting for socioeconomic conditions and population density.

The analysis combines population-weighted measures of proximity to recreational spaces with LGA-level health and socioeconomic data. It includes exploratory data analysis (EDA), ordinary least squares (OLS) regression, model diagnostics and a metropolitan–rural sensitivity analysis.

### Research questions

1. How much variation in physical activity and health outcomes is explained by socioeconomic conditions, and does population density provide additional explanatory power?
2. Does adding recreational accessibility improve model fit beyond socioeconomic conditions and population density?
3. Which accessibility dimensions, individually or jointly, provide the greatest additional explanatory power?


## Data and variables

The unit of analysis is the LGA, with 79 observations. The five age-adjusted health outcome estimates come from the **Victorian Population Health Survey (VPHS) 2023**: less than moderate physical activity (inactivity), meeting at least 150 minutes of moderate-to-vigorous physical activity per week, obesity, high or very high psychological distress, and fair or poor self-rated health.

Six recreational accessibility indicators were constructed:

| Indicator | Description |
| --- | --- |
| Park distance | Population-weighted median straight-line distance to the nearest park |
| Natural-area distance | Population-weighted median straight-line distance to the nearest natural area |
| Facility distance | Population-weighted median straight-line distance to the nearest sport or recreation facility |
| Indoor-facility distance | Population-weighted median straight-line distance to the nearest indoor facility |
| Indoor-facility share | Percentage of recreational facilities classified as indoor within each LGA |
| Sport types | Number of distinct sports recorded within each LGA |

Distances were calculated from representative points within **populated 2021 Census Mesh Blocks**, then aggregated using Mesh Block population weights. Other explanatory variables include **Socio-Economic Indexes for Areas (SEIFA) Index of Relative Socio-economic Advantage and Disadvantage (IRSAD)** and **population density**. The VPHS metropolitan–rural classification is used in the exploratory comparisons and regional sensitivity analysis.

## Data sources and availability

The table identifies source data files and whether they are tracked in Git. Files marked **No** must be obtained separately to reproduce workflows that use them. All paths are relative to the repository root.

| File in `data/` | Source | In Git? |
|---|---|---|
| `Victorian Population Health Survey 2023.xlsx`, `VPHS_BMI.xlsx`, `VPHS_chronic.xlsx`, `VPHS_wellbeing.xlsx` | [VAHI – Victorian Population Health Survey 2023](https://vahi.vic.gov.au/reports/victorian-population-health-survey-2023): data dashboard exports for the Physical activity, BMI, psychological distress and Wellbeing chapters | Yes |
| `Local Government Area, Indexes, SEIFA 2021.xlsx` | [ABS – SEIFA 2021](https://www.abs.gov.au/statistics/people/people-and-communities/socio-economic-indexes-areas-seifa-australia/2021), Data downloads | Yes |
| `LGA/LGA_POLYGON.*` | [Vicmap Admin – LGA Polygon Aligned to Property](https://discover.data.vic.gov.au/dataset/vicmap-admin-local-government-area-lga-polygon-aligned-to-property) | Yes |
| `Population_ LGA2022-2023.xlsx` | [ABS – Regional population 2022–23](https://www.abs.gov.au/statistics/people/population/regional-population/2022-23): *Population estimates and components by LGA* (Victoria = Table 2) | Yes |
| `Mesh Block Counts, 2021.xlsx` | [ABS – Census Mesh Block counts 2021](https://www.abs.gov.au/census/guide-census-data/mesh-block-counts/2021) | Yes |
| `MB_2021_AUST_SHP_GDA2020/` | [ABS – ASGS Edition 3 boundaries](https://www.abs.gov.au/statistics/standards/australian-statistical-geography-standard-asgs/edition-3-july-2021-june-2026/access-and-downloads/digital-boundary-files): *Mesh Blocks 2021, Shapefile GDA2020* | **No** (large) |
| `facilities.xlsx` | [DataVic – Sport and Recreational Facilities list](https://discover.data.vic.gov.au/dataset/sport-and-recreational-facilities-list) (sheet `wholeIFMD`) | Yes |
| `plan_zone_pprz_pcrz.gpkg` | [Vicmap Planning – Planning Scheme Zone Polygon](https://discover.data.vic.gov.au/dataset/vicmap-planning-planning-scheme-zone-polygon), PPRZ and PCRZ only | Yes |


### Restoring data not tracked by Git

**1. Mesh Block boundaries**

The national **Mesh Blocks 2021, Shapefile GDA2020** is required to rerun `02-accessibility-indicators.qmd` and is excluded from Git because of its size.

1. Download **Mesh Blocks - 2021 - Shapefile** from the **Downloads for GDA2020 digital boundary files** section of the [ABS ASGS Edition 3 digital boundary files page](https://www.abs.gov.au/statistics/standards/australian-statistical-geography-standard-asgs/edition-3-july-2021-june-2026/access-and-downloads/digital-boundary-files) by clicking **Download ZIP**. The downloaded file is `MB_2021_AUST_SHP_GDA2020.zip`.

2. Unzip it into `data/MB_2021_AUST_SHP_GDA2020/`, so that `MB_2021_AUST_GDA2020.shp` and its associated files (`.shx`, `.dbf`, `.prj`) sit directly inside this folder.

**2. Planning zones**

The repository tracks `data/plan_zone_pprz_pcrz.gpkg`, which contains only the relevant **Public Park and Recreation Zone (PPRZ)** and **Public Conservation and Resource Zone (PCRZ)** features. Its source is the [Vicmap Planning dataset](https://discover.data.vic.gov.au/dataset/vicmap-planning-planning-scheme-zone-polygon).

The original subset was obtained from the Vicmap open-data [WFS service](https://opendata.maps.vic.gov.au/geoserver/wfs), using layer `open-data-platform:plan_zone`, the filter `zone_code IN ('PPRZ','PCRZ')`, and pagination in batches of 5,000 features. The larger raw downloads in `data/vicmap_data/` are not tracked and are not required to rerun the current scripts, which read the tracked `plan_zone_pprz_pcrz.gpkg` subset.


## How to reproduce

The project uses R and Quarto.

**Required packages:**

```r
install.packages(c(
  "broom", "car", "corrplot", "dplyr", "geogrid", "ggh4x",
  "ggplot2", "janitor", "kableExtra", "knitr", "lmtest",
  "patchwork", "purrr", "readxl", "sandwich", "sf",
  "skimr", "spdep", "tibble", "tidyverse"
))
```

**Full pipeline** (needs all raw data):

```bash
quarto render 01-data-preparation.qmd
quarto render 02-accessibility-indicators.qmd
quarto render 03-eda.qmd
quarto render 04-model.qmd
quarto render report.qmd
quarto render presentation.qmd
```


## Key variables

The LGA-level analysis dataset is `clean_data/analysis_lga.csv` (one row per LGA).

| Variable | Meaning |
|---|---|
| `inactive_percent`, `activity_percent`, `obesity_percent`, `distress_percent`, `poor_health_percent` | Age-adjusted % of adults: less than moderate activity; ≥150 min/week moderate-to-vigorous activity; obesity (BMI ≥ 30); high/very high distress (K10 ≥ 22); fair/poor self-rated health. `_ll`/`_ul` = 95% CI, `_se` = SE derived from the CI |
| `SEIFA_IRSAD` | Socioeconomic advantage score (higher = more advantaged) |
| `population_density_2023` | Estimated resident population 2023 per km² |
| `med_dist_park_m`, `med_dist_nature_m`, `med_dist_facility_m`, `med_dist_indoor_m` | Population-weighted median distance (m) from residents' Mesh Blocks to the nearest park (PPRZ), natural area (PCRZ), facility and indoor facility |
| `pct_indoor_facilities`, `sport_type_count` | % of facilities that are indoor; number of sport types in the LGA |
| `region_vphs` | VPHS metropolitan–rural classification: metropolitan (31 LGAs) or rural (48 LGAs) |

## Main findings

Accessibility provided the clearest additional explanatory value for **physical inactivity**. Adding accessibility indicators increased adjusted R² from **0.612** in the SEIFA-and-density baseline to **0.713** in Model A (HC3 joint *p* < 0.001). Natural-area distance gave the largest individual improvement for inactivity (Δ adjusted R² = **0.068**) despite a near-zero unadjusted correlation. The joint contribution remained significant after regional adjustment. Comparable significant joint contributions were not found for the other four outcomes.

These are **LGA-level associations, not causal effects**.

## License

The original code developed for this project is licensed under the MIT License. See the [LICENSE](LICENSE.md) file for details.

Third-party datasets and resources remain subject to their respective licences and terms of use.