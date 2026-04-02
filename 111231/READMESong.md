# 🌊 Flood Mapping in Songkhla Province, Thailand

> **A minimal, structured, and document-based workflow for flood mapping with Sentinel-1 SAR on Google Earth Engine / Google Colab**

---

## Overview

This project maps flood inundation in **Songkhla Province** during the **November 2025 flood event** using **Sentinel-1 Synthetic Aperture Radar (SAR)** imagery processed in **Google Earth Engine (GEE)**. The workflow is designed for **Google Colab**, with emphasis on a clear methodological explanation and reproducible steps.

The explanation in this README is based mainly on the uploaded Songkhla report, which identifies Songkhla as a flood-prone coastal province with lowlands, wetlands, urban areas such as Hat Yai, and hydrologic influence from both the **Gulf of Thailand** and **Songkhla Lake basin**. These geographic conditions make the province vulnerable to flood inundation, especially during the **northeast monsoon** season. The report also specifies the exact SAR-based workflow used in the analysis: pre/post-event comparison, VV/VH ratio, speckle filtering, thresholding, masking permanent water, terrain filtering, and connected-pixel cleanup. The Lab 4 brief further requires that the README explain the **method chosen, reasons, limitations, and findings** in an organized repository format. Finally, the flood lecture notes support the physical logic that high precipitation input over low-lying terrain often leads to flooding. 

---

## 🎯 Objectives

This notebook and README were prepared to:

1. map flood extent in **Songkhla Province** during the late November 2025 event,
2. demonstrate a **remote sensing workflow** that can operate under cloudy weather conditions,
3. build a clean, reproducible pipeline in **Python + GEE** that runs on **Google Colab**, and
4. document the method in a way that is suitable for coursework and future adaptation.

---

## 🗺️ Study Area

Songkhla Province is located in southern Thailand on the eastern coast of the Malay Peninsula. According to the report, the province covers about **7,393 km²**, lies between roughly **6°17′–7°56′ N** and **100°01′–101°06′ E**, and includes coastal plains, hills, wetlands, and the **Songkhla Lake** system. The lake and its drainage network make many surrounding low-lying zones vulnerable to seasonal flooding. The report highlights that areas such as **Hat Yai** are especially exposed because they are urbanized lowlands receiving water from surrounding higher terrain before draining toward Songkhla Lake and the Gulf of Thailand. 

The same document explains that the province is influenced by a **tropical monsoon climate**, with annual rainfall around **1,800–2,500 mm**, and most heavy precipitation occurring during the **northeast monsoon** between **October and January**. This seasonal rainfall pattern is a key reason for choosing Songkhla as the study area.

---

## 💡 Why Sentinel-1 SAR?

Optical satellite imagery is often limited during flood events because cloud cover is typically dense during heavy rain. The Songkhla report therefore uses **Sentinel-1 SAR**, which records microwave backscatter and can operate **through clouds**, **day and night**, and during severe weather. This makes it well suited to rapid flood assessment.

The Chiang Rai reference document and the supporting flood-mapping literature in the uploaded files also emphasize the same logic: SAR is especially useful for flood detection because it captures backscatter responses from water surfaces and remains operational in cloudy conditions, which is critical during active flood periods.

---

## 🧱 Data Used

### Table 1. Datasets used in the workflow

| Dataset | GEE Collection / Source | Spatial Resolution | Purpose in Workflow |
|---|---|---:|---|
| Sentinel-1 SAR GRD | `COPERNICUS/S1_GRD` | 10 m | Main flood detection dataset using VV and VH backscatter |
| JRC Global Surface Water | `JRC/GSW1_4/GlobalSurfaceWater` | 30 m | Remove permanent or seasonal water bodies from flood results |
| SRTM DEM | `USGS/SRTMGL1_003` | 30 m | Compute slope and remove steep terrain unlikely to retain floodwater |
| FAO GAUL administrative boundaries | `FAO/GAUL/2015/level1` | vector | Define Songkhla Province as the Region of Interest (ROI) |

### Table 2. Sentinel-1 filtering settings

| Parameter | Value |
|---|---|
| Product | Ground Range Detected (GRD) |
| Instrument mode | IW |
| Polarization | VV, VH |
| Orbit | Descending |
| Resolution | 10 m |

### Table 3. Temporal windows used in the analysis

| Period | Date Range | Role |
|---|---|---|
| Before flood | **1 Oct 2025 – 31 Oct 2025** | Baseline condition before inundation |
| After flood | **20 Nov 2025 – 30 Nov 2025** | Flood-period condition used for change detection |

The Songkhla report states that **median composites** were generated for both periods to reduce image noise and improve stability before comparing the two periods.

---

## 🧭 Conceptual Principle

The conceptual basis of this project is straightforward:

- floods tend to occur when **water input exceeds drainage and storage capacity**,
- low-lying terrain and poorly drained urban or floodplain settings are more likely to retain water,
- during active flood events, open water generally produces a **distinct SAR backscatter response**, and
- changes between pre-flood and post-flood SAR scenes can therefore be used to map inundated pixels.

The flood lecture slides support this physical reasoning by noting that stream valleys and flat terrain are common locations for human settlement, but also that **high amounts of water often lead to flooding**. The slides also explain that precipitation varies through time and that **high inflow leads to flooding**, which supports the use of a before/after comparison framework.

---

## ⚙️ Methodology

The workflow follows the structure described in the Songkhla report, with implementation adapted for Python in Google Colab.

### 1) Define the Region of Interest (ROI)

The first step is to isolate **Songkhla Province** from the FAO GAUL administrative dataset. This boundary is used throughout the workflow to clip imagery, calculate statistics, and display results.

Why this step matters:
- it limits computation to the study area,
- it ensures that all later outputs are spatially consistent, and
- it makes the workflow transferable to another province by simply changing the ROI.

---

### 2) Retrieve Sentinel-1 SAR imagery

Sentinel-1 images are filtered using the settings listed above:
- ROI = Songkhla Province,
- instrument mode = IW,
- polarization = VV and VH,
- orbit = descending,
- resolution = 10 m,
- two date windows: before-flood and after-flood.

After filtering, a **median composite** is created for each period.

Why median composite is used:
- to reduce scene-to-scene variability,
- to suppress random noise,
- to create a more stable representation of the period rather than relying on one scene only.

---

### 3) Add the polarization ratio band (VV/VH)

The Songkhla report calculates a ratio band:

```text
VV/VH = VV / VH
```

This ratio is added to the SAR image to improve discrimination between water surfaces and nearby land cover types.

Why this helps:
- different land surfaces respond differently in VV and VH,
- open water often has a distinct backscatter contrast,
- the ratio provides an additional feature beyond the original VV and VH bands.

The Chiang Rai reference document similarly notes that composite polarization information such as **VV, VH, and VV/VH** can improve interpretation of flooded versus non-flooded surfaces.

---

### 4) Reduce speckle noise with Refined Lee filter

SAR data naturally contains **speckle noise**, which appears as granular variation and can reduce classification reliability if used directly. The Songkhla methodology therefore applies a **Refined Lee filter** to the **VH band**.

Why Refined Lee is used:
- it reduces random speckle,
- it preserves important edges better than simple smoothing,
- it improves stability in the later change-detection step.

This is especially important because flood boundaries often occur near roads, channels, settlements, and agricultural field edges, where oversmoothing could remove meaningful spatial detail.

---

### 5) Perform ratio-based change detection

The core flood-detection step compares the post-flood and pre-flood conditions using:

```text
Change Ratio = After Flood / Before Flood
```

Pixels with sufficiently large change values are considered likely inundated. In the Songkhla report, a threshold of:

```text
Change Ratio > 1.2
```

is used to define **initial flood pixels**.

Why change detection is appropriate:
- it focuses on **event-related change**,
- it reduces dependence on absolute backscatter values alone,
- it is effective when the goal is to isolate newly inundated areas.

This threshold-based logic is also consistent with the Chiang Rai reference, which explains that flood mapping can be performed through **difference/threshold approaches** after comparing radar backscatter between pre-event and flood-event images.

---

### 6) Remove permanent water bodies

Not all water-like pixels in the flood-period image represent floodwater. Rivers, lakes, wetlands, and perennial water bodies may already be present before the event. To avoid confusing permanent water with new inundation, the workflow masks water bodies using **JRC Global Surface Water**.

In the Songkhla report, areas with water occurrence for more than **five months per year** are treated as permanent/seasonal water and excluded from the flood result.

Why this step matters:
- it reduces false positives,
- it separates long-term water presence from event-based flooding,
- it makes the final map more representative of flood impact.

---

### 7) Remove steep terrain using slope

Floodwater usually accumulates in **low-relief** terrain rather than on steep slopes. The workflow therefore computes terrain slope from the **SRTM DEM** and masks out areas with:

```text
Slope > 5°
```

Why this is a sound physical filter:
- steep areas are less likely to hold standing floodwater,
- many false detections in SAR can occur in complex terrain,
- the filter helps focus the map on floodplains, lowlands, and urban basins.

This matches the geographic description of Songkhla, where flood-prone areas are concentrated in **coastal plains**, **lake-margin lowlands**, and **urban low-elevation zones**.

---

### 8) Remove isolated noise with connected-pixel filtering

Thresholded SAR results often contain very small clusters caused by noise. The Songkhla report removes such artifacts using **connected pixel filtering** and retains only clusters with at least:

```text
15 connected pixels
```

Why this helps:
- small isolated clusters are often misclassification noise,
- larger connected patches are more likely to represent real inundation,
- the final map becomes cleaner and easier to interpret.

---

### 9) Generate the final flood map

After thresholding and all masking/filtering steps, the remaining pixels are interpreted as the **final inundated area** for the flood event.

The final flood layer can then be:
- visualized on an interactive map in Colab,
- exported as raster output,
- used for spatial statistics and summary tables.

---

### 10) Calculate flood area

The Songkhla report calculates the number of flooded pixels and converts this to area using the 10 m pixel size of Sentinel-1.

### Table 4. Flood area summary reported in the document

| Parameter | Value |
|---|---:|
| Flood pixels | 644,222 pixels |
| Flooded area | 64,422,200 m² |
| Flooded area | 64.42 km² |
| Flooded area | 40,263.88 rai |

This conversion follows the basic logic:

```text
Area (m²) = number of flood pixels × pixel area
```

with pixel area = `10 × 10 = 100 m²`.

---

## 🧪 Interpretation of Results

According to the Songkhla report, the mapped flooded areas are mainly concentrated in:
- **coastal floodplains**,
- **agricultural land**,
- **low-elevation zones around Songkhla Lake**, and
- **urban regions with poor drainage**.

The document also notes that the **northern districts** show larger flooded clusters because of their flat terrain and proximity to water bodies, while **Hat Yai District** exhibits urban flooding linked to dense development and limited drainage capacity.

This pattern is physically consistent with the study-area description:
- lowland terrain is more likely to retain floodwater,
- the Songkhla Lake basin influences drainage pathways,
- coastal setting and high sea level can reduce efficient outflow,
- urban surfaces can intensify waterlogging where drainage is insufficient.

---

## 🧠 Why this method is reasonable

This workflow is intentionally simple, interpretable, and grounded in documented flood-mapping practice.

### Strengths

- **Cloud-independent observation** using SAR
- **Simple and transparent logic** through pre/post comparison
- **Physically informed filtering** with permanent-water masking and slope masking
- **Fast implementation** suitable for rapid event mapping
- **Good fit for Colab + GEE** because all core datasets are already available online

### Why each method choice is justified

| Method choice | Reason |
|---|---|
| Sentinel-1 SAR | Works in cloudy flood conditions, day or night |
| Before/after composite | Captures event-based change instead of static water only |
| VV/VH ratio | Improves separability of water versus land cover |
| Refined Lee filter | Reduces SAR speckle while preserving important boundaries |
| Threshold > 1.2 | Practical rule from the source workflow for selecting flooded pixels |
| JRC GSW masking | Prevents confusion between floodwater and permanent water bodies |
| Slope < 5° | Floodwater is more likely to persist in flatter terrain |
| Connected pixels ≥ 15 | Removes isolated noise and improves readability |

---

## ✅ Validation and credibility

The original Songkhla report is mainly a **flood extent mapping** workflow rather than a full multi-factor flood-risk model. For a coursework setting, this is still useful because it produces a physically interpretable inundation map. However, in a stricter Lab 4 framework, the README should also acknowledge that validation can be improved.

The Chiang Rai reference document provides one useful direction: it discusses checking results against external flood information and using **Intersection over Union (IoU)** as a validation concept. In practice, validation could be strengthened by comparing the mapped flood area against:
- flood boundaries from GISTDA or another agency,
- event-period optical interpretation where clouds allow,
- local reports or field observations,
- high-resolution manually interpreted water masks.

### Honest assessment

This project is reliable as a **rapid flood detection workflow**, but it is not yet a full hydrological or hydraulic simulation. It detects likely inundation from satellite-observed change, not the full causal mechanics of river discharge, drainage network capacity, or time-evolving water levels.

---

## ⚠️ Limitations

A good README should state limitations clearly.

1. **Threshold sensitivity**  
   The value `1.2` is taken from the project workflow. A different flood event, land cover type, or seasonal condition may require threshold adjustment.

2. **Backscatter ambiguity**  
   Some non-water surfaces can also change in radar response due to soil moisture, roughness, vegetation structure, or agricultural activity.

3. **Permanent-water masking is imperfect**  
   Wetlands and seasonally inundated zones may be difficult to separate perfectly from event floodwater.

4. **No direct ground truth in the notebook**  
   The current workflow is strongest as a rapid mapping product. Accuracy would be more defensible with independent validation data.

5. **Urban complexity**  
   In dense urban areas, buildings and radar geometry can complicate the backscatter response, so urban flooding may be under- or over-detected in some locations.

---

## 🔭 Suggestions for extension

To improve the project further, future versions could add:

- rainfall forcing from **ERA5**,
- distance-to-river or drainage-network variables,
- land use / impervious surface data,
- hydrological accumulation or flow-direction factors,
- threshold sensitivity testing,
- validation against an external flood polygon,
- export of GeoTIFF outputs and summary figures.

These additions would make the project closer to the broader Lab 4 expectation of multi-factor spatial modeling, while keeping the SAR flood-detection core.

---

## ▶️ How to run on Google Colab

### 1. Open the notebook
Open `Songkhla_Colab.ipynb` in Google Colab.

### 2. Install dependencies
Run the installation cell, for example:

```python
!pip -q install earthengine-api geemap
```

### 3. Authenticate Earth Engine
If needed, authenticate with your Google account and initialize Earth Engine with your Google Cloud project:

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

### 4. Run cells in order
Run all notebook cells from top to bottom:
- imports,
- authentication,
- ROI setup,
- Sentinel-1 filtering,
- ratio calculation,
- Refined Lee filtering,
- change detection,
- masking,
- area calculation,
- map display.

### 5. Review outputs
Inspect:
- pre-flood image,
- post-flood image,
- change layer,
- final flood map,
- total inundated area summary.

---

## 📁 Suggested project structure

```text
GE338-Lab-4/
├── Songkhla_Colab.ipynb
├── README.md
├── conceptual_framework.md
└── figures/
```

---

## 📌 Key takeaway

This project shows that a carefully filtered **Sentinel-1 SAR change-detection workflow** can map flood extent in Songkhla quickly and clearly. It is especially suitable for severe flood periods when optical imagery is limited by cloud cover. The mapped pattern is consistent with the province’s documented geography: **low-lying terrain, lake-basin influence, coastal exposure, and urban drainage constraints**, especially around **Hat Yai**.

---

## References used to structure this README

- Songkhla flood mapping report
- Lab 4 Geographic Modeling brief
- Chiang Rai SAR flood mapping reference
- GE316/GE315 flood lecture notes

