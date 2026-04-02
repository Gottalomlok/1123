# Data Inventory

## Project
Mars on Earth: การระบุพื้นที่แห้งแล้งและพืชพรรณต่ำในภาคตะวันออกเฉียงเหนือของประเทศไทยด้วยข้อมูลรีโมตเซนซิง

| Dataset | Source / Collection | Temporal Coverage Used | Variable / Band | Used for | Limitation | Feasibility Check |
|---|---|---|---|---|---|---|
| MODIS Vegetation Index | MODIS/061/MOD13Q1 | 2001–2024 | NDVI | Vegetation condition, Low Vegetation | scale factor needed, 250 m | Notebook checks collection size and annual composite |
| MODIS ET | MODIS/061/MOD16A2GF | 2001–2024 | ET | Annual evapotranspiration for AI | coarse spatial resolution, unit conversion | Notebook checks collection size and export preview |
| GPM IMERG Monthly | NASA/GPM_L3/IMERG_MONTHLY_V07 | 2001–2024 | precipitation | Annual rainfall | monthly aggregation required | Notebook aggregates monthly to annual rainfall |
| MODIS LST | MODIS/061/MOD11A2 | 2001–2024 | LST_Day_1km | Supporting climate context | 1 km resolution | Notebook builds annual mean LST |
| SRTM DEM | USGS/SRTMGL1_003 | static | elevation | Terrain context | no time variation | Notebook clips DEM to study area |
| ISAN Shapefile | Google Drive / local shapefile | current | polygon boundary | Study area boundary | geometry / CRS must be checked | Notebook validates CRS and converts to EPSG:4326 |

## Notes
- The analysis uses **2001–2024** as the practical study period so that all major datasets overlap more reliably.
- The shapefile is loaded from Google Drive in Colab, which allows the workflow to run without uploading a GEE asset first.
- Export outputs include PNG maps, CSV summary tables, and GeoTIFF raster layers.
