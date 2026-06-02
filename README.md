# Wildfire Risk Indicator

This project has the goal of creating an Environmental Indicator. The Final Indicator is a bivariate Matrix consisting of two axis. One is a vulnerability axis. This one was premade in QGIS and is provided as one final _.tif_ that can be downloaded and put into the raw data folder. The other one is the Ecological value axis, that is based on the MESLI framework, decsribed below. This one was made in the notebook *ecological_value.ipynb*. In another notebook *final_indicator.ipynb*, the two axis are combined to the bivariate matrix. A third notebook *name* provides extra context, regarding the policy of the targeted countries, providing a potential framework for making the indicator performance-based. 

##  Setup Instructions

### Step 1: Clone the Repository
First, download this repository to your local Desktop (or use another location of your preference) and enter the directory:

```bash
cd ~/Desktop
git clone https://github.com/mlengenfelder/GEO888
cd GEO888
```

### Step 2: Create and Activate the Environment

```bash
conda env create -f environment.yml
conda activate wildfire_indicator
```

### Step 3: Initialize the Directory Structure
* Open your Jupyter environment, open the notebook *ecological_value.ipynb*, select ecological-value as your kernel, and run the very first setup cell (Section: _0. Setup_).
* Running this cell automatically creates the local folder path structure (data/raw/, data/processed/, outputs/, etc.) that is missing from the repository.

### Step 4: Download and Position the Raw Data
Now that the paths exist, download the raw data. Access the links below and place all input data into the raw data folder, that you just created!

* GEO888/Data/Raw/

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
* Open the *ecological_value.ipynb* notebook.
* Ensure your notebook kernel is set to _ecological-value_.
* Run the remaining cells sequentially from top to bottom (Cell-wise is highly recommended).
* When you completed running the 1st notebook, run the 2nd notebook *final_indicator.ipynb*.
* After that, run the 3rd and final notebook *script_performance_based.ipynb*.
* All outputs (interactive maps, plots, and metrics) will be saved in outputs/ and data / processed. 

# 1. Notebook: Ecological Value / MESLI (ecological_value.ipynb)

This notebook creates the ecological value axis as a forest-only MESLI raster.
It is the notebook to run **FIRST**!

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

# 2. Notebook: Final Indicator (final_indicator.ipynb)

## Inputs
.....

# 3. Notebook: NLP Processing or EU-Report for assesing policy measures across europe

## Inputs
.....