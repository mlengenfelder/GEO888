# Ecological Value / MESLI

This project creates the ecological value axis as a forest-only MESLI raster.
The complete workflow is implemented in `notebook.ipynb`; there is no separate
processing script.

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

## Setup in VS Code

```bash
conda env create -f environment.yml
conda activate ecological-value
```

Expected input files:

```text
Data/Raw/BAWS_Map_2020.tif
Data/Raw/Biomass_Map_2020.tif
Data/Raw/gHM_Clipped.tif
Data/Raw/pm25_avg19.tif
Data/Raw/U2018_CLC2018_V2020_20u1.tif
Data/Raw/U2018_CLC2018_V2020_20u1.tif.vat.dbf
Data/Raw/Natura2000_end2024.gpkg
```

## Run

Open `notebook.ipynb` in VS Code and run the cells from top to bottom.

## Outputs

```text
Data/Processed/corine_1000m_epsg3035.tif
Data/Processed/forest_mask_1000m_epsg3035.tif
Data/Processed/forest_type_score_1000m_epsg3035.tif
Data/Processed/natura2000_binary_1000m_epsg3035.tif
Data/Processed/baws_1000m_epsg3035.tif
Data/Processed/biomass_1000m_epsg3035.tif
Data/Processed/ghm_1000m_epsg3035.tif
Data/Processed/pm25_1000m_epsg3035.tif
Data/Processed/baws_norm_1000m_epsg3035.tif
Data/Processed/biomass_norm_1000m_epsg3035.tif
Data/Processed/ghm_inverted_1000m_epsg3035.tif
Data/Processed/pm25_inverted_norm_1000m_epsg3035.tif
Data/Processed/provisioning_score_1000m_epsg3035.tif
Data/Processed/cultural_score_1000m_epsg3035.tif
Data/Processed/regulating_score_1000m_epsg3035.tif
Data/Processed/biodiversity_score_1000m_epsg3035.tif
Data/Processed/MESLI_ecological_value_raw_1000m_epsg3035.tif
Data/Processed/MESLI_ecological_value_1000m_epsg3035.tif
Data/Processed/indicator_summary.csv
Data/Processed/output_grid_validation.csv
Data/Processed/MESLI_interactive_map.html
```

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

It writes validation tables to:

```text
Data/Processed/indicator_summary.csv
Data/Processed/output_grid_validation.csv
```
