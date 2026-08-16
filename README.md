# Assam Flood 2022: Multi-Sensor Inundation & Impact Assessment

An interactive Google Earth Engine (GEE) application designed to detect, map, and assess the impact of the devastating June 2022 floods in Assam, India. The tool leverages Sentinel-1 Synthetic Aperture Radar (SAR) for cloud-penetrating flood detection, Sentinel-2 optical imagery for contextual baselines, and Google Dynamic World for land use/land cover (LULC) impact assessment.

---

## 📌 Overview

In mid-2022, severe monsoon rainfall triggered catastrophic flooding across the Brahmaputra and Barak river basins in Assam. Because flood events are characterized by dense monsoon cloud cover, optical remote sensing is often rendered ineffective. 

This project utilizes dual-polarization **Sentinel-1 SAR (C-band)** change detection to identify inundated areas, masks out permanent surface water and topographic shadows, and calculates the direct impact on **cropland** and **urban/built-up** areas. The results are presented via a synchronized 4-panel interactive Earth Engine dashboard.

---

## 🛰️ Datasets Used

| Dataset | Earth Engine Snippet | Purpose |
| :--- | :--- | :--- |
| **Sentinel-1 SAR GRD** | `COPERNICUS/S1_GRD` | Pre- and post-flood radar imagery (IW mode, VH polarization, 10m) |
| **Sentinel-2 MSI (Level-2A)** | `COPERNICUS/S2_SR` | Pre-flood cloud-free RGB optical baseline composite |
| **JRC Global Surface Water** | `JRC/GSW1_3/GlobalSurfaceWater` | Seasonality layer used to mask out permanent water bodies |
| **NASADEM Digital Elevation** | `NASA/NASADEM_HGT/001` | Slope calculation (< 12°) to eliminate radar shadow misclassifications |
| **Dynamic World V1** | `GOOGLE/DYNAMICWORLD/V1` | Real-time LULC masks for `crops` and `built` (urban) exposure analysis |

---

## ⚙️ Methodology & Pipeline

1. **Temporal Filtering:**
   * **Baseline Period:** `2022-01-01` to `2022-02-28` (Dry season pre-flood)
   * **Flood Event Period:** `2022-06-10` to `2022-06-23` (Peak inundation)
2. **SAR Preprocessing & Speckle Reduction:**
   * Filtered for `VH` polarization and `DESCENDING` orbit passes.
   * Applied a circular focal mean filter ($r = 50\text{ m}$) to suppress radar speckle noise.
3. **Change Detection & Thresholding:**
   * Difference map calculation: `after_filtered.subtract(before_filtered)`
   * Backscatter difference threshold: $> 1.25\text{ dB}$
4. **Refinement & False-Positive Removal:**
   * **Permanent Water Mask:** JRC Water Seasonality $\ge 6$ months masked out.
   * **Isolated Pixel Filter:** Connected pixel count threshold $\ge 8$ pixels.
   * **Topographic Mask:** Terrain slope $> 12^\circ$ masked out using NASADEM to eliminate hill shadows.
5. **Impact & Exposure Mapping:**
   * Overlay inundation mask onto **Dynamic World** probability bands (`crops` and `built`).
   * Zonal statistics via `reduceRegion` to compute total flooded area in **hectares (ha)**.

---

## 🖥️ Interactive 4-Panel UI Layout

The application utilizes `ui.Map.Linker` to synchronously pan and zoom across 4 split-view maps:


* **Map 1 (Flood Zone):** Focus areas with 1,000m buffer zones around key coordinates.
* **Map 2 (After Flood):** Detected raw inundation layer (`#118ab2`).
* **Map 3 (Affected Cropland):** Crop exposure overlay (`#aacc00`) on Sentinel-2 true-color composite.
* **Map 4 (Affected Urban):** Built-up / infrastructure exposure overlay (`#d00000`) on Sentinel-2 composite.

---

## 🚀 How to Run

1. Open the [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
2. Create a new script and define your Area of Interest as an `ee.Geometry` or import a shapefile named `geometry` (or `aoi`).
3. Paste the contents of `flood_assam_2022.js` into the editor.
4. Click **Run**.
5. Pan and zoom around Assam to explore synced views and inspect the bottom-left layer legends.

---

## 🔧 Key Parameters for Customization

```javascript
// Temporal Windows
var before_start = '2022-01-01';
var before_end   = '2022-02-28';
var after_start  = '2022-06-10';
var after_end    = '2022-06-23';

// SAR Detection Parameters
var polarization         = "VH";      // VH is sensitive to flood surface changes
var pass_direction       = "DESCENDING";
var difference_threshold = 1.25;      // Adjust based on sensitivity requirements
var smoothing_radius     = 50;        // Speckle smoothing radius (meters)
