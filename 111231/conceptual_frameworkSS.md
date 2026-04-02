# Conceptual Framework — Flood Mapping in Songkhla Province, Thailand

## 1. Problem Statement

This project studies the **late-November 2025 flood event in Songkhla Province** and asks:

> **Where were the flood-affected areas in Songkhla Province during the November 2025 event, and how can these areas be detected from radar imagery using a geographically justified workflow?**

The result must be spatial and map-based, which matches the requirement of Lab 4.

---

## 2. Why Songkhla?

Songkhla is an appropriate flood case study because the uploaded report describes the province as:
- a coastal province influenced by the **Gulf of Thailand** and **Songkhla Lake**,
- containing extensive **lowlands and coastal plains**,
- affected by the **northeast monsoon** during the main flood season,
- and especially vulnerable in urban and low-lying areas such as **Hat Yai**.

These conditions make the province highly suitable for flood mapping with remote sensing.

---

## 3. Scientific Logic Behind the Model

Flooding occurs when water input exceeds the ability of the land surface and drainage system to absorb, store, or convey it away. In the lecture notes on flooding, precipitation is described as a major driver because water input varies through time, and high inflow leads to flooding.

For the Songkhla case, the conceptual logic is:

1. **Heavy monsoon rainfall** increases water input.
2. **Low-lying, flat terrain** is more likely to retain or accumulate water.
3. **Radar backscatter changes** between before- and after-event images can indicate newly inundated surfaces.
4. **Permanent water bodies** must be removed because they are normal background water, not floodwater.
5. **Steep terrain** is less likely to hold widespread standing water.
6. **Small isolated radar artifacts** should be removed because they are less likely to represent real flood patches.

The workflow therefore combines **remote sensing evidence** with **terrain-based plausibility** and **post-processing cleanup**.

---

## 4. Factors Used in the Workflow

Although the implemented method is event-detection oriented, it still includes multiple meaningful geographic factors:

### Factor 1 — SAR backscatter before flood
- Source: Sentinel-1 SAR
- Role: Represents pre-event surface condition
- Reason: Provides the baseline against which flood-related change can be measured

### Factor 2 — SAR backscatter after flood
- Source: Sentinel-1 SAR
- Role: Represents event-period condition
- Reason: Captures the altered radar response caused by water inundation

### Factor 3 — Polarization ratio (VV/VH)
- Source: Derived from Sentinel-1
- Role: Enhances discrimination between water and non-water surfaces
- Reason: Water surfaces have distinct radar scattering behavior

### Factor 4 — Permanent water mask
- Source: JRC Global Surface Water
- Role: Removes normal water bodies
- Reason: Prevents confusion between permanent water and floodwater

### Factor 5 — Terrain slope
- Source: SRTM DEM
- Role: Masks steep terrain
- Reason: Standing floodwater is more plausible on flatter terrain than on steep slopes

### Factor 6 — Connected pixel structure
- Source: Derived from classified flood mask
- Role: Removes isolated noisy detections
- Reason: Real flood patches are typically spatially coherent

---

## 5. Model Logic

The model is based on the following sequence:

```text
Sentinel-1 before-event composite
        +
Sentinel-1 after-event composite
        ↓
Change ratio detection
        ↓
Thresholding (Change Ratio > 1.2)
        ↓
Mask permanent water (JRC GSW)
        ↓
Mask steep slopes (> 5° from SRTM)
        ↓
Remove small clusters (< 15 connected pixels)
        ↓
Final flood map
```

This is a geographic model because it combines several spatial layers and applies physically meaningful constraints.

---

## 6. Why These Decision Rules Were Chosen

### Threshold = 1.2
This threshold is taken from the uploaded Songkhla report and used to define significant change in the ratio image.

### Permanent water > 5 months/year
This criterion is used to separate floodwater from surfaces that are normally water-covered.

### Slope > 5°
This rule removes terrain where widespread flood retention is less plausible.

### Connected pixels ≥ 15
This step reduces small noisy patches and improves the readability of the final map.

---

## 7. Sensitivity Consideration

The most influential parameter in the implemented workflow is the **change ratio threshold**.  
A simple sensitivity test can be done by varying it by ±20%:

- Low threshold: **0.96**
- Baseline threshold: **1.20**
- High threshold: **1.44**

This helps identify:
- robust flood zones,
- uncertain boundary areas,
- and the degree to which the final map depends on the classification cutoff.

---

## 8. Validation Plan

### Validation currently possible
- Compare spatial pattern with known flood timing and affected lowland areas
- Visually inspect coherence with expected hydrologic behavior
- Review whether results avoid permanent water and steep terrain

### Stronger validation recommended
- Compare against official flood polygons from GISTDA or related agencies
- Use Sentinel-2 or other optical imagery when cloud-free scenes are available
- Create sample points for accuracy assessment
- Compute metrics such as confusion matrix, IoU, precision, and recall if reference data are available

---

## 9. Strengths of the Model

- Works under cloudy monsoon conditions
- Uses physically interpretable factors
- Province-scale implementation is computationally practical
- Reproducible in Google Colab with Earth Engine
- Easy to update for future flood events

---

## 10. Limitations of the Model

- It detects flood extent, not flood depth
- It does not explicitly model rainfall, drainage capacity, or tidal influence
- Threshold-based classification may be sensitive in ambiguous areas
- Urban radar response can be complex
- Validation is incomplete without independent flood reference data

---

## 11. Appropriate Scale of Use

This model is best suited for:
- **district-scale**
- **provincial-scale**
- and broad **regional event mapping**

It is less appropriate for:
- building-scale interpretation,
- parcel-scale damage estimation,
- or detailed engineering assessment.

---

## 12. Conclusion

The conceptual framework for this project treats flooding in Songkhla as the result of **event-driven water change observed by SAR imagery**, refined by **terrain constraints**, **permanent-water masking**, and **post-classification cleanup**. This makes the final flood map geographically defensible, transparent, and appropriate for a Lab 4 submission focused on spatial modeling and critical interpretation.
