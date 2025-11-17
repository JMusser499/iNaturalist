# Connecticut Native Flora Flowering Phenology

Comprehensive analysis of flowering phenology and abundance patterns for Connecticut tracheophytes using iNaturalist observations from across New England (2015-2024).

## Overview

This R Markdown project processes iNaturalist community science observations to generate detailed visualizations of flowering phenology across New England flora, with particular emphasis on Connecticut native species. The analysis combines temporal phenology data with abundance classifications to provide a comprehensive view of when and where plants flower throughout the year.

## Output Products

The script generates four multi-page PDF documents:

### 1. Species Phenology Barplots
**File:** `phenology_CT_tracheophytes_complete.pdf`

Multi-page faceted barplots organized taxonomically (family → genus → species), showing:
- Weekly flowering observations across all 52 weeks
- Total observation counts (gray line)
- Sparse data handling (solid bars, light bars, or points based on observation counts)
- Color-coded abundance categories
- Native status indicated by asterisk (*)
- One genus per page with up to 9 species per page

### 2. CT Native Species Ridge Plots  
**File:** `ct_native_flowering_ridgeplots.pdf`

Ridge plots of flowering phenology for Connecticut native species, featuring:
- Species ordered by peak flowering time (earliest to latest)
- Density distributions showing flowering timing across New England
- Scientific names, families, and common names
- Consistent color gradient by week (viridis palette)
- 25 species per page in portrait orientation

### 3. Family Overview Table
**File:** `family_overview_table.pdf`

Summary table showing species counts per family across:
- CT native species count
- Abundance categories (common, uncommon, rare, NE-only)
- Total species per family

### 4. Rare CT Native Species Table
**File:** `rare_ct_native_species_flowering.pdf`

Detailed phenology table for rare Connecticut native species (≤10 CT observations), including:
- Peak flowering month and week
- Earliest flowering date
- Flowering duration (weeks)
- Total flowering observations
- Connecticut observation counts

## Key Features

### Phenological Metrics

- **Peak flowering time**: Calculated as weighted mean (center of mass) of flowering observations
- **Flowering duration**: 10th-90th percentile window captures central 80% of flowering period
- **Earliest flowering date**: First recorded flowering observation across all years
- **Temporal resolution**: 52-week calendar year (avoids ISO week complications)

### Abundance Classification

Species are classified into five categories based on observation counts in Connecticut and New England:

| Category | CT Observations | NE Observations | Description |
|----------|----------------|-----------------|-------------|
| Connecticut common | >50 | — | Well-documented in CT |
| Connecticut uncommon | 11-50 | — | Moderately documented in CT |
| Connecticut rare | 1-10 | — | Poorly documented in CT |
| New England | 0 | >10 | Present in NE but not CT |
| New England rare | 0 | 1-10 | Rare across New England |

### Data Quality Visualization

Sparse data handled through visual encoding:
- **Solid bars** (≥15 flowering observations): High confidence
- **Light bars** (5-14 flowering observations): Moderate confidence  
- **Points only** (<5 flowering observations): Low confidence, exploratory

### Native Status

Connecticut native species identified using BONAP (Biota of North America Program) data via the Connecticut Botanical Society checklist. Native species marked with asterisk (*) throughout visualizations.

## Requirements

### R Packages

```r
# Core data manipulation
library(readr)
library(dplyr)
library(tidyr)
library(purrr)
library(stringr)
library(lubridate)

# Visualization
library(ggplot2)
library(ggridges)
library(ggforce)
library(forcats)
library(cowplot)

# Tables and grids
library(gridExtra)
library(grid)

# API access (for common names)
library(httr)
library(jsonlite)

# Document formatting
library(knitr)
library(BiocStyle)  # optional, for HTML output
```

### Data Files Required

1. **GBIF Darwin Core Archive**: iNaturalist observations for New England tracheophytes
   - Download from GBIF (https://www.gbif.org/)
   - Unzip to directory specified in script (default: `dwca_ne_angiosperms`)

2. **Connecticut Native Species Checklist**: `CT Flora Checklist 8-1-14.csv`
   - Source: Connecticut Botanical Society
   - Contains native status (N) for CT species

3. **Common Names Cache** (auto-generated): `inat_common_names_cache.csv`
   - Created automatically by script via iNaturalist API
   - Cached to minimize API calls

## Usage

### Basic Workflow

1. **Prepare data files:**
   ```r
   # Place GBIF download in working directory
   # Ensure CT native checklist CSV is available
   ```

2. **Configure parameters:**
   ```r
   # In Step 0: Setup and Configuration
   unzipped <- "dwca_ne_angiosperms"  # Path to GBIF data
   ```

3. **Run analysis:**
   ```r
   # In RStudio:
   rmarkdown::render("inat_flowering_times_improved2.Rmd")
   
   # Or via command line:
   Rscript -e "rmarkdown::render('inat_flowering_times_improved2.Rmd')"
   ```

4. **Output files:**
   - PDFs generated in working directory
   - HTML report (optional) with detailed summaries

### Customization Options

#### Adjust Ridge Plot Density
```r
# In Step 15: Create Multi-Page Ridge Plot PDF
species_per_page <- 25  # Increase for more compact, decrease for more detail
bandwidth = 1.5         # Ridge smoothing (1.0 = detailed, 2.5 = smooth)
scale = 2               # Ridge overlap (lower = less overlap)
```

#### Modify Abundance Thresholds
```r
# In Step 0: Setup and Configuration
min_ct_obs_rare <- 10L        # Threshold for CT rare
min_ct_obs_uncommon <- 50L    # Threshold for CT uncommon
min_ne_obs_rare <- 10L        # Threshold for NE rare
```

#### Change Color Schemes
```r
# Ridge plots (Step 15):
scale_fill_viridis_c(option = "magma")  # Options: viridis, magma, plasma, inferno, cividis

# Barplots (Step 10):
# Edit scale_fill_manual() values
```

## Methodology

### Data Processing Pipeline

1. **Data Import**: Read GBIF occurrence data (Darwin Core Archive format)
2. **Temporal Filtering**: Restrict to last 10 years (2015-2024)
3. **Taxonomic Standardization**: Collapse subspecies/varieties to species level
4. **Phenology Extraction**: Parse iNaturalist reproductive condition annotations
5. **Abundance Classification**: Categorize by observation counts
6. **Native Status Integration**: Join with CT Botanical Society checklist
7. **Temporal Binning**: Aggregate observations into 52-week bins
8. **Statistical Summaries**: Calculate peak flowering, duration, earliest date
9. **Visualization**: Generate multi-page PDFs

### Phenology Annotation

Flowering observations identified from iNaturalist's "reproductive condition" field:
- **Flowering**: "flowers", "flowering", "flower buds"
- Annotations are user-contributed and may have varying quality
- Only annotated observations used for phenology (non-annotated excluded)

### Peak Flowering Calculation

Peak flowering time calculated as **weighted mean** of weekly observations:

```r
peak_week = weighted.mean(week, n_flowering)
```

This "center of mass" approach is more robust than simple maximum, especially for:
- Multi-modal distributions (species with multiple flowering peaks)
- Noisy data with observation gaps
- Extended flowering periods

### Data Quality Considerations

- **Observation bias**: iNaturalist observations biased toward:
  - Accessible locations (trails, parks, roadsides)
  - Charismatic/easily identified species
  - Peak flowering times (when plants are most noticeable)
  
- **Geographic coverage**: New England observations used for all species to increase sample size, but may not reflect CT-specific phenology

- **Temporal bias**: Recent years (2020-2024) over-represented due to iNaturalist growth

## File Structure

```
.
├── inat_flowering_times_improved2.Rmd    # Main analysis script
├── CT Flora Checklist 8-1-14.csv         # Native status data
├── dwca_ne_angiosperms/                  # GBIF data directory
│   ├── occurrence.txt                    # Occurrence records
│   └── ...
├── inat_common_names_cache.csv           # Auto-generated cache
├── phenology_CT_tracheophytes_complete.pdf
├── ct_native_flowering_ridgeplots.pdf
├── family_overview_table.pdf
├── rare_ct_native_species_flowering.pdf
└── README.md
```

## Example Output

### Ridge Plot Features
- **Top to bottom**: Species ordered by earliest to latest peak flowering
- **Left to right**: January (left) to December (right)
- **Color gradient**: Purple (early season) → Yellow (late season)
- **Ridge height**: Density of observations at that time

### Barplot Features
- **Bars**: Flowering observations (colored by abundance category)
- **Gray line**: Total observations including non-flowering
- **Opacity**: Visual indicator of data quality
- **Facets**: One species per panel, organized by genus

## Data Sources

### Primary Data
- **iNaturalist Research-Grade Observations**
  - Accessed via GBIF (Global Biodiversity Information Facility)
  - Geographic scope: Connecticut, Maine, Massachusetts, New Hampshire, Rhode Island, Vermont
  - Taxonomic scope: Tracheophytes (vascular plants)
  - Temporal scope: Last 10 years
  - License: CC0, CC-BY, CC-BY-NC (depending on observer preferences)

### Reference Data
- **Connecticut Native Status**: Connecticut Botanical Society flora checklist
  - Based on BONAP (Biota of North America Program)
  - Last updated: August 1, 2014

### Common Names
- **iNaturalist API**: Species common names retrieved via iNaturalist taxonomy
  - Cached locally to minimize API calls
  - Preferred English common names where available

## Citation

If you use this analysis or visualizations in publications, please cite:

**Data:**
```
GBIF.org (2024). GBIF Occurrence Download. https://doi.org/[your_download_DOI]
```

**iNaturalist:**
```
iNaturalist contributors, iNaturalist (2024). iNaturalist Research-grade 
Observations. iNaturalist.org. Occurrence dataset https://doi.org/10.15468/ab3s5x 
accessed via GBIF.org on [date].
```

**Connecticut Native Status:**
```
Connecticut Botanical Society (2014). Flora of Connecticut Checklist. 
Available from: http://www.ct-botanical-society.org/
```

## Author

**Jacob M. Musser**
- Evolutionary biologist specializing in developmental neuroscience and early animal evolution
- Botanical expertise in Connecticut flora identification

## License

This analysis code is released under MIT License. Please note that:
- iNaturalist observation data has mixed licenses (CC0, CC-BY, CC-BY-NC)
- CT Botanical Society checklist usage should be acknowledged
- Generated visualizations should cite data sources appropriately

## Contributing

Suggestions for improvements are welcome! Potential areas for enhancement:
- Additional phenological metrics (onset, offset, duration variability)
- Climate/weather data integration
- Year-over-year phenology shift analysis
- Comparison with historical herbarium records
- Interactive web-based visualizations

## Acknowledgments

- **iNaturalist community**: Thousands of observers contributing phenology observations
- **GBIF**: Data aggregation and standardization infrastructure
- **Connecticut Botanical Society**: Native species determinations
- **R Community**: Package developers (tidyverse, ggplot2, ggridges, etc.)

## Version History

- **v1.0** (2024-11): Initial release with four PDF outputs
  - Barplots by family/genus
  - Ridge plots by peak flowering time
  - Family overview tables
  - Rare species tables

---

For questions or issues, please open a GitHub issue or contact the author.
