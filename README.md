# Wind Farm Suitability
A spatial decision-support workflow for identifying optimal wind farm locations using multi-criteria analysis and real-world validation.

### Why this study

Wind farm siting is not a single-variable problem. It is a spatial optimisation problem involving competing technical, environmental, and economic factors. Selecting locations based only on wind speed leads to unrealistic or non-deployable sites. 
Wind energy is scalable and cost competitive. The site selection is one of the most important things that directly impacts project feasibility and cost. Poor siting can lead to high transmission cost, construction difficulty and regulatory rejection. 
This study addresses that gap using a structured GIS–MCDA framework.

**Approach:**
* Use **Analytic Hierarchy Process (AHP)** for weight derivation
* Integrate all factors spatially using **GIS-based weighted overlay**
* Apply **constraint masking** to eliminate infeasible land
* Use **continuous (fuzzy) standardisation** instead of rigid classes

**Methodology Justification:**
* Handles **multi-dimensional decision-making**
* Reduces **subjective bias** via pairwise comparison
* Preserves data fidelity using **continuous suitability surfaces**
* Aligns with established GIS–MCDA practices in wind siting literature

This makes the model relevant for:
* early-stage site screening
* infrastructure planning
* renewable energy policy support
--------------------------------------------------------------------------------------------------
**STUDY AREA**

This study focuses on **Tamil Nadu and Kerala (India)**, selected to represent both high wind development zones and complex terrain conditions within a single modelling framework.

**Why this region**

Tamil Nadu is one of India’s most mature wind energy markets, with established wind corridors (e.g., Coimbatore–Tirunelveli) and strong grid infrastructure. This makes it ideal for validating whether the model aligns with real-world deployment patterns.

Kerala, in contrast, introduces significant terrain complexity due to the Western Ghats. Despite having wind potential, development is limited, making it useful for testing how terrain and constraints influence suitability.

**Key characteristics**

* Mature wind infrastructure (Tamil Nadu)
* Complex mountainous terrain (Kerala)
* Coastal + inland wind variability
* Existing transmission network for grid connectivity

**Spatial setup**

The study area was created by merging administrative boundaries of both states and used as a clipping extent for all datasets.

Coordinate system: **WGS 1984 UTM Zone 43N (EPSG:32643)**
This ensures accurate distance and area calculations required for spatial analysis.

--------------------------------------------------------------------------------------------------
**INPUT FACTORS**


**Core factors**
* **Wind resource** — Global Wind Atlas (100 m wind speed → wind power index)
* **Terrain** — Copernicus DEM (slope, terrain ruggedness)
* **Infrastructure**

  * distance to roads (OSM)
  * distance to power lines (OSM)

**Constraint layers (excluded from analysis)**

* settlements (buffered)
* protected areas (WDPA)
* land use / land cover (ESA WorldCover)
* airports (government dataset)

All datasets were projected to **WGS 1984 UTM Zone 43N (EPSG:32643)** and clipped to the study area.

-----------------------------------------------------------------------------------------------------
**CONSTRAINTS**

Constraint layers were used to **exclude non-developable areas** before final site selection. These were applied as a binary mask (1 = allowed, 0 = excluded).

**Exclusion criteria**

* **Settlements** — 1 km buffer (noise, safety, shadow flicker)
* **Protected areas** — full exclusion (environmental regulations)
* **Land use / land cover** — removed:

  * built-up areas
  * water bodies
  * wetlands and mangroves
* **Airports** — 5 km buffer (aviation safety)

**Method**

All constraint layers were converted to raster format and combined into a single mask:

* individual masks → binary (0/1)
* combined using multiplication
* applied to final suitability raster

**Result**

* 0 → excluded land
* 1 → available land

This ensures that final outputs represent only **physically and legally feasible locations**.

---------------------------------------------------------------------------------------------------------------
**METHODS**

The suitability model was built using a GIS-based MCDA workflow combining fuzzy standardisation, AHP weighting, and weighted overlay.

**Standardisation (fuzzy suitability layers)**

All factors were transformed to a common **[0–1] scale** to enable comparison.

* **Wind power** — increasing suitability (percentile-based scaling)
* **Slope** — decreasing linear function (0–20° range)
* **Terrain ruggedness** — percentile-based (P10–P90 scaling)
* **Distance to grid** — decreasing linear (0–10 km)
* **Distance to roads** — decreasing linear (0–25 km)

Full equations and implementation details are provided in /docs/methodology.md.

**AHP weighting**

Criterion weights were derived using the Analytic Hierarchy Process (AHP).

* Pairwise comparison matrix constructed in Google Sheets
* Normalisation → eigenvector (weights)
* Consistency Ratio (CR) = **0.033 (< 0.1, acceptable)**

Final weights:

* Wind: 0.517
* Grid: 0.237
* Slope: 0.103
* Ruggedness: 0.097
* Roads: 0.046

*Note: The AHP matrix and calculations are included in the repository (see `/docs` or spreadsheet file).*

**Weighted overlay**

Final suitability computed using weighted linear combination:

* weighted sum of all standardised layers
* implemented in ArcGIS Raster Calculator
* output: continuous suitability raster

**Constraint application**

* Combined constraint mask applied post-overlay
* removes all excluded areas from final surface

**Site extraction**

High-suitability zones were converted into candidate sites:

* smoothing (majority filter)
* extraction of top classes (4 & 5)
* region grouping → contiguous zones
* raster → polygon conversion
* area filtering (≥ 5 km²)

Final output: **222 candidate wind farm sites**

---------------------------------------------------------------------------------------------------------------------------
**FINAL RESULTS & VALIDATION**

The final suitability map represents a continuous surface of wind farm suitability, classified into five categories from very low to very high.

**Key results**

* High suitability zones are concentrated in **Tamil Nadu wind corridors**
* Large portions of Kerala show lower suitability due to **terrain constraints**
* After applying constraints, only **feasible land** is retained for analysis

**Classification**

The final raster was classified into 5 classes:

* 1 — Very Low
* 2 — Low
* 3 — Moderate
* 4 — High
* 5 — Very High

High and very high zones were used for site extraction.

**Site extraction output**

* Total candidate sites: **222**
* Minimum site size: **≥ 5 km²**
* Derived from contiguous high-suitability regions

**Validation**

A validation layer of existing wind turbines was created using OpenStreetMap (Overpass Turbo).

* ~12,800+ turbine points extracted
* Converted, projected, and clipped to study area
* Overlaid on final suitability map

**Observation**

* Existing turbines are **strongly concentrated in high (4) and very high (5) suitability zones**
* Minimal presence in low-suitability areas

This indicates that the model successfully captures **real-world wind deployment patterns**, supporting its reliability for preliminary site selection.

-------------------------------------------------------------------------------------------------------------------------
