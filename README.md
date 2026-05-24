# Ecological Value / MESLI

This project creates the ecological value axis as a forest-only MESLI raster.
The complete workflow is implemented in `notebook.ipynb`; there is no separate
processing script.

All available input rasters are transformed to a shared grid:

- CRS: `EPSG:3035`
- resolution: `1000 m`
- extent/grid: derived from CORINE
- output folder: `Data/Processed`

## Inputs

| Category | Indicator | Unit / processing | Dataset | Use |
|---|---|---|---|---|
| Forest area | Extent of analysis | Forest yes/no | CORINE Land Cover 2018 + VAT DBF | Final mask only |
| Biodiversity | Species richness / endangered species | Natura2000 habitat yes/no | Natura2000 | Binary 0/1 layer |
| Biodiversity | Value of forest | Forest type proxy | CORINE Land Cover 2018 | Broad-leaved `311/23 = 1.00`, mixed `313/25 = 0.75`, coniferous `312/24 = 0.50` |
| Provisioning | Timber production | 0-1 normalized BAWS | BAWS Map 2020 | Provisioning category |
| Cultural | Recreation / naturalness | `1 - gHM` | Global Human Modification | Cultural category |
| Regulating | Climate regulation / biomass | 0-1 normalized biomass | Biomass Map 2020 | Regulating sub-indicator |
| Regulating | Air quality burden | `1 - PM2.5_norm` | PM2.5 2019 | Regulating sub-indicator |

The CORINE DBF is an attribute table, not a spatial mask. The notebook reads it
to document that CLC classes `311`, `312`, and `313` are stored as raster values
`23`, `24`, and `25`. The actual forest mask is created directly from the
CORINE raster.

## Setup in VS Code

```bash
conda env create -f environment.yml
conda activate ecological-value
python -m ipykernel install --user --name ecological-value --display-name "Python (ecological-value)"
code .
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
- identical width, height, and transform
- score values between `0` and `1`
- final MESLI values only on forest pixels

It writes validation tables to:

```text
Data/Processed/indicator_summary.csv
Data/Processed/output_grid_validation.csv
```
