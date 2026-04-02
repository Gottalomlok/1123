# 🌊 Flood Mapping in Songkhla Province, Thailand Using Sentinel-1 SAR
> **GE338 Lab 4 — Geographic Modeling**  
> Multi-Criteria / spatially explicit flood analysis workflow adapted for **Google Colab** and written as a submission-ready repository.

---

## ✨ Project Overview

This project analyzes the **November 2025 flood event in Songkhla Province** using **Sentinel-1 Synthetic Aperture Radar (SAR)** data on **Google Earth Engine (GEE)**. The workflow is designed for a notebook environment and emphasizes a clear geographic modeling logic, careful preprocessing, and transparent discussion of assumptions and limitations.

Although the computational workflow uses **before–after flood detection** from SAR imagery, the project is framed as a **geographic modeling exercise** in line with Lab 4: define the problem, justify the factors used, explain the model logic, evaluate sensitivity, and discuss validation in a scientifically honest way.

The explanation in this README is based primarily on the uploaded Songkhla flood report, the Lab 4 brief, and the supporting flood-mapping references you provided.

---

## 🎯 Research Question

**How can Sentinel-1 SAR imagery and supporting geographic datasets be used to detect flood-affected areas in Songkhla Province during the late-November 2025 flood event, and how reliable is the resulting flood map for geographic interpretation and future flood-risk applications?**

This question is spatial, map-based, and directly aligned with the Lab 4 requirement that the result must be answerable through a geographic output.

---

## 🗺️ Study Area

The study area is **Songkhla Province**, located in southern Thailand on the eastern coast of the Malay Peninsula. The province has diverse terrain including coastal plains, urban areas, hills, wetland environments, and the **Songkhla Lake** system. Several low-lying areas, especially around **Hat Yai**, are highly exposed to flooding because runoff from surrounding uplands converges into flatter zones before draining toward Songkhla Lake and the Gulf of Thailand.

According to the uploaded report, Songkhla has:
- lowlands and coastal plains that are prone to inundation,
- watercourses and catchment areas feeding urban zones,
- strong influence from the **northeast monsoon** during October–January,
- annual rainfall roughly **1,800–2,500 mm**, and
- frequent flood events in low-elevation and poorly drained locations.

These characteristics make Songkhla an appropriate province for flood mapping with radar imagery.

---

## 🧠 Conceptual Framework Summary

Flooding in this project is interpreted as a spatial phenomenon controlled by both **event conditions** and **surface characteristics**.

At the conceptual level:

- **Heavy rainfall / monsoon conditions** increase water input into the basin.
- **Low-lying terrain** retains water more easily than steep terrain.
- **Permanent water bodies** must be removed so they are not falsely labeled as floodwater.
- **Backscatter change in SAR imagery** helps identify newly inundated surfaces.
- **Noise filtering and cluster filtering** are necessary because SAR data contain speckle and isolated classification artifacts.

This means the final flood map is not produced from a single threshold alone. Instead, it results from a chain of geographically reasoned decisions: **image filtering → ratio calculation → flood threshold → hydrologic plausibility masks → post-processing refinement**.

A fuller conceptual framework is provided in [`conceptual_framework.md`](./conceptual_framework.md).

---

## 🛰️ Data Used

### Table 1. Input datasets

| Dataset | GEE ID / Source | Spatial Resolution | Role in Workflow | Why It Is Used |
|---|---|---:|---|---|
| Sentinel-1 SAR GRD | `COPERNICUS/S1_GRD` | 10 m | Main flood detection dataset | Radar can observe the surface through cloud cover and during rainy conditions |
| JRC Global Surface Water | `JRC/GSW1_4/GlobalSurfaceWater` | 30 m | Permanent water masking | Prevents rivers/lakes that normally contain water from being misclassified as floodwater |
| SRTM DEM | `USGS/SRTMGL1_003` | 30 m | Slope calculation | Helps remove steep terrain where standing floodwater is less likely |
| FAO GAUL administrative boundary | `FAO/GAUL/2015/level1` | vector | Region of interest (ROI) | Defines the boundary of Songkhla Province for analysis |

### Table 2. Sentinel-1 selection settings

| Parameter | Value |
|---|---|
| Product | GRD |
| Instrument mode | IW |
| Polarizations | VV, VH |
| Orbit pass | Descending |
| Spatial resolution | 10 m |

### Table 3. Analysis periods

| Period | Date Range | Purpose |
|---|---|---|
| Before flood | 1 Oct 2025 – 31 Oct 2025 | Reference condition before major inundation |
| After flood | 20 Nov 2025 – 30 Nov 2025 | Flood-event condition during/after inundation |

The report states that **median composites** were used for both periods to reduce noise and improve image stability. This is a sound step because SAR scenes can vary across acquisition dates.

---

## ⚙️ Methodology

## 1) Define the region of interest (ROI)

The workflow begins by extracting the administrative boundary of **Songkhla Province** from the FAO GAUL dataset. This boundary becomes the **ROI** for all subsequent filtering, masking, and export operations.

Why this matters:
- It limits computation to the study area.
- It ensures the reported flood area corresponds only to Songkhla.
- It makes the workflow reproducible and comparable with the province-scale framing in the report.

---

## 2) Load and filter Sentinel-1 SAR imagery

The notebook filters Sentinel-1 images using:
- the ROI,
- the instrument mode `IW`,
- both `VV` and `VH` polarizations,
- `DESCENDING` orbit,
- and the two event periods described above.

Why Sentinel-1 is appropriate:
- Flood events often occur under heavy cloud cover.
- Optical imagery may fail during monsoon periods.
- SAR is weather-independent and can capture surface water conditions during and after major rainfall events.

The report specifically justifies Sentinel-1 because microwave radar can penetrate clouds and is suitable for flood monitoring.

---

## 3) Create median composites for before and after periods

All valid scenes in each period are reduced to a **median composite**.

Why median composite is important:
- SAR data are noisy by nature.
- A single acquisition may contain local anomalies.
- Median compositing creates a more stable representative surface for each period.

This helps make the change detection step more reliable.

---

## 4) Add polarization ratio (VV/VH)

A new band is created:

```text
VV/VH = VV / VH
```

Why this ratio is used:
- Open water typically has distinct radar backscatter characteristics.
- Combining VV and VH can improve separation between flooded and non-flooded surfaces.
- The uploaded reference materials support the use of VV, VH, and ratio-based combinations for water/flood analysis.

This ratio is not the only decision variable, but it adds interpretive value and improves discrimination.

---

## 5) Reduce speckle noise with Refined Lee filtering

SAR imagery inherently contains **speckle noise**, which can produce grainy patterns and false detections. To address this, the workflow applies a **Refined Lee filter** to the VH band.

Why Refined Lee is used:
- It suppresses random speckle while preserving spatial edges.
- It is widely used in radar image preprocessing.
- Flood boundaries become more coherent after filtering.

This is a critical step because noisy input produces noisy flood masks.

---

## 6) Perform change detection between before and after conditions

The main detection logic uses a ratio-based change comparison:

```text
Change Ratio = After Flood / Before Flood
```

Pixels with sufficiently strong change are interpreted as newly flooded areas.

Why change detection is appropriate:
- The goal is to identify **inundation caused by the event**, not simply map all water.
- Comparing before and after conditions helps isolate flood-related change.
- This is more meaningful than classifying only one date.

---

## 7) Apply the flood threshold

The initial flood mask is created using a threshold of:

```text
Change Ratio > 1.2
```

Why 1.2 is used:
- This threshold comes directly from the Songkhla report.
- It represents a practical cutoff for separating significant radar backscatter change from background variation.
- In a teaching / lab context, using the documented threshold improves reproducibility.

Important note:
- This value is not universal.
- A different event, land cover, incidence angle, or hydrologic context may require recalibration.

---

## 8) Remove permanent and seasonal water bodies

The workflow masks permanent water using **JRC Global Surface Water**. Areas that show water occurrence for more than five months per year are treated as permanent water and excluded from the flood result.

Why this is necessary:
- Lakes, reservoirs, large rivers, and stable wetlands may appear water-covered in both normal and flood conditions.
- Without masking, normal water would be incorrectly interpreted as floodwater.
- This step improves event-specific interpretation.

For Songkhla, this is particularly important because the province includes extensive water environments such as the **Songkhla Lake** system.

---

## 9) Remove steep terrain using slope from SRTM

Slope is calculated from the SRTM DEM, and areas with slope greater than **5 degrees** are masked out.

Why slope filtering is used:
- Floodwater is more likely to accumulate in low-slope and low-lying terrain.
- Steep terrain is less likely to support widespread standing water.
- This adds a geomorphological constraint to the detection result.

This step is especially useful for preventing false positives on rough or mountainous surfaces.

---

## 10) Remove isolated noise patches with connected-pixel filtering

After thresholding and masking, small isolated clusters are removed. The workflow keeps only flood clusters with at least **15 connected pixels**.

Why this step is needed:
- Small scattered patches often result from residual radar noise or local artifacts.
- Real floodwater usually forms spatially coherent patches.
- Connected-pixel filtering increases the visual and analytical reliability of the final map.

This is a post-classification cleanup step and improves interpretability.

---

## 11) Produce the final flood map

The output of the preceding steps is the final flood layer, which represents flood-affected areas during the late-November 2025 event.

In the source report, the flood result was summarized as:

### Table 4. Flood area summary

| Metric | Value |
|---|---:|
| Flood pixels | 644,222 |
| Flood area (m²) | 64,422,200 |
| Flood area (km²) | 64.42 |
| Flood area (rai) | 40,263.88 |

These values were derived from 10 m Sentinel-1 pixels, where each pixel covers 100 square meters.

---

## 🧪 Sensitivity Analysis

Lab 4 requires a sensitivity analysis. Even though the base Songkhla workflow is threshold-based, sensitivity can still be assessed by varying the most influential decision parameter: the **change ratio threshold**.

### Parameter tested
- Baseline threshold: **1.2**
- Lower scenario: **0.96**  (-20%)
- Higher scenario: **1.44** (+20%)

### Why threshold sensitivity matters
The threshold controls how aggressively the workflow labels a backscatter change as floodwater. Small threshold changes can produce:
- broader flood extents in marginally affected zones,
- fewer detections in noisy or ambiguous surfaces,
- and different levels of agreement with visually expected inundation patterns.

### Expected interpretation
- **Robust areas**: flood zones detected under all thresholds; likely to represent strong inundation signals.
- **Uncertain areas**: flood zones that appear only at lower thresholds; likely to be boundary zones, shallow water, mixed pixels, or noisy surfaces.
- **Conservative core flood extent**: areas surviving the high threshold; likely to be the most certain flood patches.

### What this tells us
If the map changes only slightly when the threshold varies by ±20%, confidence in the method increases. If the result changes dramatically, the workflow is highly sensitive and should be interpreted with caution.

---

## ✅ Validation Strategy

Lab 4 asks for validation with real data. In this Songkhla workflow, there are two levels of validation to discuss.

### A. Validation available in the project
The result is supported by:
- temporal logic (before vs after imagery),
- physically meaningful masks (permanent water, slope),
- spatial coherence filtering,
- and consistency with the documented flood event.

### B. Stronger validation recommended
A stronger validation would compare the final flood map against:
- official flood extent from GISTDA or disaster agencies,
- independently interpreted flood polygons,
- MODIS flood products where available,
- or manually checked reference samples from Sentinel-2 / high-resolution imagery during clear-sky windows.

### Honest assessment
The current workflow is strong for **rapid event mapping**, but it is not a full hydrologic simulation and does not include:
- river stage,
- drainage network capacity,
- rainfall accumulation grids,
- tide interaction,
- or field-verified inundation depth.

Therefore, validation should be presented as **reasonable but incomplete**, which is scientifically more credible than overstating accuracy.

---

## 📌 Why This Workflow Makes Geographic Sense

This workflow is not just a technical sequence of code. Each step has a geographic rationale:

| Step | Geographic meaning |
|---|---|
| ROI selection | Defines the province-scale analytical unit |
| SAR filtering | Chooses event-relevant radar observations |
| Median composite | Stabilizes the surface signal |
| VV/VH ratio | Enhances water discrimination |
| Refined Lee filter | Reduces SAR noise while preserving spatial structure |
| Change ratio | Detects event-driven water change |
| Permanent water masking | Separates normal water from floodwater |
| Slope masking | Applies terrain plausibility |
| Connected-pixel filtering | Removes isolated artifacts and improves map coherence |

This combination makes the final map easier to justify in a Lab 4 setting.

---

## 💻 How to Run on Google Colab

## 1. Open the notebook
Open `Songkhla_Colab.ipynb` in Google Colab.

## 2. Install required packages
Run the installation cell:

```python
!pip -q install earthengine-api geemap
```

## 3. Authenticate Earth Engine
Use a Google account with Earth Engine access and initialize with your Google Cloud project:

```python
import ee
import geemap

PROJECT = "your-google-cloud-project-id"

try:
    ee.Initialize(project=PROJECT)
except Exception:
    ee.Authenticate()
    ee.Initialize(project=PROJECT)
```

## 4. Run cells in order
Run the notebook from top to bottom:
1. initialization,
2. ROI definition,
3. Sentinel-1 preprocessing,
4. flood detection,
5. masking and cleaning,
6. area calculation,
7. visualization,
8. export.

## 5. Export results
To export the flood raster to your Google Drive:

```python
export_task = ee.batch.Export.image.toDrive(
    image=flood_final.toByte(),
    description="Songkhla_Flood_Map_2025",
    folder="GEE_exports",
    fileNamePrefix="songkhla_flood_map_2025",
    region=roi.geometry(),
    scale=10,
    maxPixels=1e13,
    fileFormat="GeoTIFF"
)
export_task.start()
```

---

## 🔍 Required Lab 4 Reflection Questions

## 1) If ground-truth data were available, what accuracy might this model achieve? Why?

A realistic expectation is **moderate to high accuracy**, but not perfect. The method is strong because it uses:
- weather-independent SAR data,
- before–after comparison,
- permanent water masking,
- terrain filtering,
- and patch cleaning.

However, accuracy would likely drop in areas with:
- mixed land cover,
- urban backscatter complexity,
- shallow or transient water,
- radar shadow / geometric effects,
- and wetlands that are difficult to separate from floodwater.

Therefore, a cautious expectation would be **useful operational accuracy for province-scale event mapping**, but not survey-grade precision at parcel level.

---

## 2) What important factors were not included, and why?

Several important factors were not explicitly included in the current workflow:

| Missing factor | Why it matters | Why it was not included |
|---|---|---|
| Rainfall accumulation | Direct driver of flood generation | Not used in the original Songkhla report workflow |
| Drainage / river network | Controls water routing and overflow | Would require additional hydrologic preprocessing |
| Land use / impervious surface | Influences runoff and water retention | Beyond the scope of the base SAR flood-detection workflow |
| Tidal influence / sea level | Important in coastal and lake-connected settings | Not available in the implemented workflow |
| Soil / infiltration capacity | Affects runoff generation | Requires external thematic data and more modeling assumptions |
| Flood depth | Important for hazard severity | Not derivable from this detection workflow alone |

These omissions do not make the map invalid, but they do limit its interpretation.

---

## 3) At what scale does this model work best, and why?

This workflow works best at the **district to provincial scale**.

Why:
- Sentinel-1 at 10 m is detailed enough for regional flood patterns.
- The masking and threshold logic are appropriate for broad inundation mapping.
- Province-scale summary statistics are meaningful and defensible.

It is less reliable at:
- **parcel or building scale**, because SAR noise, mixed pixels, and urban complexity can strongly affect interpretation.

So the best answer is:
- **most appropriate:** district / provincial scale  
- **less appropriate:** very fine cadastral or infrastructure-scale analysis

---

## 4) If the model were updated once per year, which steps would need repeating and which would remain fixed?

### Steps that must be repeated
- Select new event dates
- Download / filter new Sentinel-1 imagery
- Build new before–after composites
- Recalculate change ratio
- Re-run thresholding, masks, and post-processing
- Recompute flood area statistics
- Reassess sensitivity and validation

### Steps that may remain fixed
- Core workflow logic
- Boundary selection method
- Permanent water masking logic
- Slope masking method
- Connected-pixel filtering logic
- export / visualization structure

### Steps that may need partial adjustment
- threshold value,
- date windows,
- map layout,
- and validation datasets.

This means the workflow is reusable, but event-specific parameters still require attention.

---

## ⚠️ Limitations

1. The threshold value of **1.2** is event-specific and should not be assumed to work universally.  
2. SAR flood detection can struggle in complex urban areas.  
3. Permanent wetlands and seasonally wet surfaces may still cause ambiguity.  
4. The model detects flood extent, not flood depth or damage severity.  
5. Validation is limited unless independent reference data are added.

---

## 🌱 Future Improvements

- Add rainfall accumulation from ERA5 or GPM
- Add drainage / river proximity factors
- Compare multiple thresholds quantitatively
- Validate with official flood polygons
- Produce uncertainty maps from threshold sensitivity
- Compare province-wide results with district-focused analyses in Hat Yai

---

## 📁 Suggested Repository Structure

```text
GE338-Lab-4/
├── Songkhla_Colab.ipynb
├── conceptual_framework.md
├── README.md
└── figures/
```

---

## 📚 Main Sources Used in This Submission

1. **Songkhla flood report** — for study area, dates, Sentinel-1 settings, VV/VH ratio, Refined Lee filtering, threshold 1.2, permanent water masking, slope masking, connected-pixel filtering, and flood area summary.  
2. **Lab 4 brief** — for conceptual framework, validation, sensitivity analysis, and README reflection requirements.  
3. **Supporting flood-mapping references** — for methodological grounding of SAR-based flood detection, ratio logic, and threshold-based classification.  
4. **Lecture notes on flood processes** — for the physical logic linking rainfall, water input, and flooding.

---

## 📝 Final Conclusion

This project demonstrates that **Sentinel-1 SAR imagery** combined with **geographic filtering steps** can produce a practical and interpretable flood map for **Songkhla Province** during the November 2025 event. The method is especially suitable for cloudy monsoon-season conditions where optical imagery may fail.

The main strength of the workflow is its simplicity, reproducibility, and geographic interpretability. Its main limitation is that it remains an **event-detection workflow** rather than a complete hydrologic model. For this reason, the final map should be interpreted as a strong province-scale flood extent estimate rather than a perfect ground-truth representation.

Even with these limitations, the workflow is appropriate as a **submission-ready Lab 4 project** because it clearly explains the problem, justifies the data and processing steps, discusses uncertainty, and reflects honestly on validation and scale.
