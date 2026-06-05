# Wildfire Risk Indicator

This project has the goal of creating an Environmental Indicator. The Final Indicator is a bivariate Matrix consisting of two axis. One is a vulnerability axis. This one was premade in QGIS and is provided as one final `.tif` that can be downloaded and put into the raw data folder. The other one is the Ecological value axis, that is based on the MESLI framework, decsribed below. This one was made in the notebook `ecological_value.ipynb`. In another notebook `final_indicator.ipynb`, the two axis are combined to the bivariate matrix. A third notebook `script_performance_based.ipynb` provides extra context, regarding the policy of the targeted countries, providing a potential framework for making the indicator performance-based. 

The overarching research question for this project is:
> _How can we measure wildfire risk in Europe and track change over time considering ecological value, vulnerability and policy goals to prioritize protection of important ecosystems?_

# Setup Instructions

<details>

<summary>Show Setup Instructions</summary>

## Step 1: Clone the Repository
First, download this repository and enter the directory:

> [!TIP]
> You can either use this command, or change the directory you want the repository to be placed in (change this: `cd ~/Desktop`). 
> Also make sure to run the commands separate!

```bash
cd ~/Desktop
git clone https://github.com/mlengenfelder/GEO888
cd GEO888
```

## Step 2: Create and Activate the Environment

```bash
conda env create -f environment.yml
conda activate wildfire_indicator
```

## Step 3: Initialize the Directory Structure
* Open your Jupyter environment, open the notebook `ecological_value.ipynb`, select *wildfire_indicator* as your kernel, and run the very first setup cell (Section: _0. Setup_).
* Running this cell automatically creates the local folder path structure (`data/raw/`, `data/processed/`, `outputs/`, etc.) that is missing from the repository.

## Step 4: Download and Position the Raw Data
Now that the paths exist, download the raw data. Access the links below and place all input data into the raw data folder, that you just created!

* `GEO888/Data/Raw/`

[Download here](https://drive.google.com/drive/folders/1jIK69oq6_iNJsSKrOb4NRa5fp5jVXjcp?usp=share_link)

Expected input files:

```text
Data/Raw/BAWS_Map_2020.tif
Data/Raw/Biomass_Map_2020.tif
Data/Raw/gHM_Clipped.tif
Data/Raw/pm25_avg19.tif
Data/Raw/U2018_CLC2018_V2020_20u1.tif
Data/Raw/U2018_CLC2018_V2020_20u1.tif.vat.dbf
Data/Raw/Natura2000_end2024.gpkg
Data/Raw/final_vulnerability_layer.tif
Data/Raw/Forestfires_report_EU.pdf
```

## Execution Order
* Open the `ecological_value.ipynb` notebook.
* Ensure your notebook kernel is set to *wildfire_indicator*.
* Run the remaining cells sequentially from top to bottom (Cell-wise is highly recommended).
* When you completed running the 1st notebook, run the 2nd notebook `script_performance_based.ipynb`.
* After that, run the 3rd and final notebook `final_indicator.ipynb`.
* All outputs (interactive maps, plots, and metrics) will be saved in `outputs/` and `data / processed`. 

</details>

# 1. Notebook: Ecological Value 
Name: `ecological_value.ipynb`

This notebook creates the ecological value axis as a forest-only MESLI raster.
It is the notebook to run **FIRST**!

<details>

<summary>Show details for 1st notebook</summary>

## 

All available input rasters are transformed to a shared grid:

- CRS: `EPSG:3035`
- Resolution: `1000 m`
- Extent/Grid: derived from CORINE
- Output folder: `Data/Processed`

## Inputs

| Category | Indicator | Operationalization | Dataset | Source |
|---|---|---|---|---|
| Forest area | Extent of analysis | Forest yes/no | CORINE Land Cover 2018 | Forest mask (Copernicus)|
| Biodiversity | Species richness | Natura2000 habitat yes/no | Natura2000 | European Environment Agency (EEA)  |
| Biodiversity | Value of forest | Forest type proxy | CORINE Land Cover 2018 | Lozano et al., 2025 |
| Provisioning | Timber production | Ton per ha of timber production  | Forest Biomass Available for Wood Supply 2020 | Avitabile, V. (2023) via Figshare |
| Cultural | Recreation / Naturalness | Continuous 0-1 metric (proportion of landscape modified) | Global Human Modification (gHM)| Kennedy, C. et al. (2018) via Figshare |
| Regulating | Climate regulation / Biomass | Ton per ha of dry aboveground forest biomass density | Biomass Map 2020 | Avitabile, V. (2023) via Figshare Collection |
| Regulating | Air quality burden | Microscopic airborne particles (< 2.5μm) | European air quality data for 2019 | European Environment Agency (EEA) via EEA SDI Portal |

## MESLI Formula

MESLI uses equal weighting at category level:

```text
Provisioning = BAWS_norm
Cultural = 1 - gHM
Regulating = mean(Biomass_norm, PM25_inverted_norm)
Biodiversity = mean(Natura2000_binary, ForestType_CORINE_score)

MESLI_raw = mean(
  Provisioning,
  Cultural,
  Regulating,
  Biodiversity
)

MESLI_forest_only = MESLI_raw * forest_mask
MESLI_normalized = minmax(MESLI_forest_only within forest mask)
```

`MESLI_ecological_value_raw_1000m_epsg3035.tif` stores the category-weighted
MESLI before the final observed min-max normalization. The main output
`MESLI_ecological_value_1000m_epsg3035.tif` is normalized to the full observed
`0-1` range inside the forest mask and is the recommended layer for combining
with a later vulnerability axis.

Outside CORINE forest classes `311`, `312`, and `313`, all final score layers
and MESLI are set to NoData. In this specific CORINE raster these forest classes
are stored as internal raster values `23`, `24`, and `25`.

## Validation

The notebook checks that all output rasters have:

- CRS `EPSG:3035`
- `1000 m` pixel size
- Identical width, height, and transform
- Score values between `0` and `1`
- Final MESLI values only on forest pixels

</details>

# 2. Notebook: Policy-Coverage across Europe 
Name: `script_performance_based.ipynb`

This notebook provides contextual, performance-based framing for the wildfire risk indicator by analysing how comprehensively each European country addresses wildfire in its national policies.
It is the notebook to run **SECOND**, after the preceding notebook has completed.

<details>

<summary>Show details for 2nd notebook</summary>

##

The EU Forest Fire Report PDF is parsed country by country, and each country's text section is searched for a set of policy-relevant keywords. The resulting classification is exported as a tabular file and as a spatial layer that is consumed as an optional overlay in notebook 2's interactive map.

- Output folder: `Data/Processed/` and `Outputs/`

## Inputs

| File | Description | Source |
|---|---|---|
| `Data/Raw/Forestfires_report_EU.pdf` | EU forest fire report with national wildfire policy descriptions | EFFIS / EU Commission |

## Keyword Search

The notebook splits each country's text into sentences and checks for the presence of the following policy-related terms:

| Keyword | Description |
|---|---|
| Prevention | Mentions of preventive measures |
| Adaptation | Climate adaptation strategies |
| Strategy | Formal national wildfire strategies |
| Research activities | Funding or execution of wildfire research |
| Wildfire management | Active management frameworks |
| Awareness campaigns | Public education and outreach |
| Information campaigns | Information dissemination programs |

## Classification Formula

```text
Yes_Count = number of keywords found in country text

Group = "Low"    if Yes_Count ≤ 1
Group = "Medium" if Yes_Count ≤ 2 (and > 1)
Group = "High"   if Yes_Count > 2
```

Each country receives a binary `Yes / No` per keyword, a total `Yes_Count`, and a three-tier `Group` classification.

## Outputs

| File | Description |
|---|---|
| `Data/Processed/country_texts/<Country>.txt` | Extracted plain-text section per country from the EU report |
| `Data/Processed/keyword_classification3.xlsx` | Full keyword matrix with Yes/No flags, Yes_Count, and Group per country |
| `Data/Processed/policy_layer.gpkg` | Country polygons with policy scores joined, in `EPSG:3035` |
| `Outputs/Wildfire_policy_coverage.png` | Choropleth map of wildfire-policy coverage across Europe |

</details>

# 3. Notebook: Final Indicator 
Name: `final_indicator.ipynb`

This notebook combines the two indicator axes — _ecological-value_ and _vulnerability_ — into the final, bivariate wildfire risk matrix.

It is the notebook to run **THIRD**, after `ecological_value.ipynb` and `script_performance_based.ipynb` has completed successfully.

<details>

<summary>Show details for 3rd notebook</summary>

##

Both input rasters are aligned to a shared grid (derived from the forest mask) before classification:

- CRS: `EPSG:3035`
- Resolution: `1000 m`
- Extent/Grid: derived from forest mask
- Output folder: `Outputs/`

## Inputs for the Vulnerability raster

The vulnerability layer was pre-processed in QGIS and Python and is provided as a single, ready-to-use `final_vulnerability_layer.tif`.

| Variable | Operationalization | Dataset | Source |
|---|---|---|---|
| Distance to nearest surface water | Euclidian distance to nearest water pixel | CORINE Land Cover 2018 | EEA, 2018 |
| Ecoregion sensitivity | Sensitivity scores 0-1 | Ecoregions2017 | Dinerstein et al., 2017 |
| Distance to nearest fire station | Bivariate: 10 km buffer around fire stations (0 or 1) | OpenStreetMap | OSM, 2026 |
| Historical fire occurrences | Historic Fire yes / no (0 or 1) | European Forest Fire Information System | EFFIS team, 2026 |
| Climate Favorability | Normalization (0-1) and mean of daily values 2020–2025 | Fire danger index | Copernicus Climate Change Service, 2019 |

## Bivariate Classification

Each forest pixel is classified into one of four quadrants using Fisher-Jenks breaks (k=2) applied independently to the MESLI and vulnerability distributions:

```text
thresh_EV  = FisherJenks(MESLI_forest_pixels, k=2)[0]
thresh_V   = FisherJenks(Vuln_forest_pixels,  k=2)[0]

Class 1 → Low EV,  Low V   → Low priority     (grey)
Class 2 → Low EV,  High V  → Medium priority  (orange)
Class 3 → High EV, Low V   → Medium priority  (blue)
Class 4 → High EV, High V  → Critical         (red)

bivariate = (ev_class * 2 + v_class + 1)
bivariate_forest_only = bivariate * forest_mask
```

Both axes are min-max normalised to `0–1` before thresholding. Pixels outside the shared forest mask are set to NoData.

## Outputs

| File | Description |
|---|---|
| `Outputs/MESLI_Vulnerability_compared.png` | Side-by-side static overview of both input axes |
| `Outputs/Wildfire_bivariate_matrix.png` | Static map of the four-class bivariate risk matrix |
| `Outputs/Wildfire_interactive_map.html` | Interactive Folium map with toggleable layers (bivariate matrix, MESLI, vulnerability, policy coverage) |
| `Data/Processed/policy_layer.gpkg` | Country-level policy layer joined from notebook 3, used as an optional overlay |

</details>