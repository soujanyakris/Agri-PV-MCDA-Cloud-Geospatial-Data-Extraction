# Agri-PV-MCDA-Cloud-Geospatial-Data-Extraction
This repository demonstrates a **cloud based** approach to geospatial data procurement. Instead of downloading static, multi-gigabyte global datasets, I built a dynamic pipeline using the Google Earth Engine (GEE) Python API to extract 10m-resolution cropland data specifically for the state of Karnataka, India.

**🚀 The Challenge**

Standard Land Use/Land Cover (LULC) datasets (like ESA WorldCover) are massive (~17GB+). Downloading and clipping these manually is inefficient. This workflow uses cloud-computation to:

- Filter the global 2021 ESA WorldCover 10m dataset.

- Clip the data to the Karnataka state boundary.

- Isolate "Class 40" (Cropland) for Agrivoltaic suitability analysis.

- Export a optimized, analysis-ready GeoTIFF.

**🛠️ Tech Stack**

Language: Python 3.x

Cloud Platform: Google Earth Engine

Libraries: * geemap: For interactive mapping and GEE simplified exports.

earthengine-api: The core communication link to Google Cloud.

matplotlib: For data integrity visualization.

**🛠️ Setup & Infrastructure**

#### 1. Google Cloud Configuration

To bypass local storage limits, I initialized a cloud-native environment:

Project Creation: Created a dedicated project in the Google Cloud Console named karnataka-agripv-project.

API Activation: Enabled the Earth Engine API to allow Python to communicate with Google's geospatial data catalog.

Registration: Registered the project for Non-Commercial / Academic use to access the Community Tier.

#### 2. Environment Requirements

The analysis is performed in a Jupyter Notebook using the following stack:

```
Bash
pip install earthengine-api geemap numpy matplotlib
```

#### 🛰️ Data Pipeline Process

#### Step 1: Authentication & Initialization

We authenticate the session using OAuth2, linking the local Jupyter environment to the Google Cloud project ID.

```
Python
ee.Authenticate()
ee.Initialize(project='karnataka-agripv-project')
```

#### Step 2: Spatial Filtering

Using the ESA WorldCover V200 collection, I filtered for the Cropland classification.

Data Source: ESA/WorldCover/v200

Target Class: 40 (Cropland)

Region of Interest (ROI): Karnataka State 

```
# Load Karnataka boundary
india_states = ee.FeatureCollection("FAO/GAUL/2015/level1")
karnataka = india_states.filter(ee.Filter.eq('ADM1_NAME', 'Karnataka'))

# Load ESA WorldCover
worldcover = ee.ImageCollection("ESA/WorldCover/v200").first()

# Cropland mask (Class 40)
cropland_mask = worldcover.eq(40).clip(karnataka)
```

#### Step 3: Optimized Export

To ensure the file was manageable for local MCDA (Multi-Criteria Decision Analysis), I exported the result at a 150m scale. This reduced the file size from ~80MB to a highly performant ~15MB GeoTIFF without losing the regional farm patterns.

```
# Define output filename
output_file = "karnataka_cropland_new.tif"

print(f"Starting export for {output_file}...")

# Export locally using geemap

geemap.ee_export_image(
    cropland_mask,
    filename=output_file,
    scale=150,  
    region=karnataka.geometry(), 
    file_per_band=False
)
```

#### 📊 Key Results

Output File: karnataka_cropland_new.tif

Data Type: Binary Raster (1 = Suitable Cropland, 0 = Non-Agricultural/Exclusion Zone)

Utility: This file serves as the "Agriculture Layer" for the final Agrivoltaic Suitability Model.
