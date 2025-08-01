# Dataset Sourcing Documentation:   Sentinel2 Satellite-Derived Crop Classification Training Data for Amini Decoding field Challenge Dataset One 

This document outlines the methodology for sourcing the training dataset for the **Geospatial Foundation Model Crop Classification Challenge**. The core approach focuses on extracting Sentinel-2 time-series spectral data from **specific, well-known geographical regions** for each target crop within Côte d'Ivoire.

## 1. Main Approach & Regional Focus 🎯

The central strategy for generating this training data is to simulate ground truth by extracting satellite time-series information from **precisely defined geographical polygons** corresponding to known cultivation areas of specific crops. This method provides labeled examples of spectral signatures over time, directly supporting the challenge's goal of classifying crop types from space.

Instead of traditional field surveys, we leverage **pre-identified coordinates** that serve as representative points within predominant cultivation zones for each crop in Côte d'Ivoire. These specific polygons, derived from these general areas, act as our "ground truth" for data extraction. The objective is to create a robust, labeled time-series dataset that captures the unique spectral characteristics of each target crop over multiple growing seasons, enabling the training of a classifier on top of GeoFM embeddings.


## 2. Data Sources & Polygon Definition 🌍

The training data is derived from:

* **Sentinel-2 Level-2A Surface Reflectance:** Accessed via the `COPERNICUS/S2_SR_HARMONIZED` Image Collection in Google Earth Engine (GEE).
* **Geographical Polygons (Ground Truth Regions):** These polygons are the foundation of our labeling process. They are defined within specific, high-production cultivation areas in Côte d'Ivoire, centered around the following representative coordinates and their corresponding regions:
    * **Rubber:** Polygons are defined within the **Comoé District** of Côte d'Ivoire, near the border with Ghana. This area is characterized by forests, rivers, and agricultural land, with the coordinates -6.306578958987097, 6.741564292145348 serving as a reference point near the Comoé River in the Sud-Comoé region.
    * **Palm:** Polygons are defined within the **Lagunes District** of southeastern Côte d'Ivoire, near the border with Comoé District. This low-lying coastal area features lagoons, wetlands, and tropical forests, with the coordinates -3.0555949286292305, 5.216843915897 serving as a reference point near the Ébrié Lagoon system, where palm oil plantations are common.
    * **Cocoa:** Polygons are defined within the **Montagnes District** of western Côte d'Ivoire, near the border with Liberia. This region is part of the Guinean forest-savanna mosaic, characterized by dense forests, hills, and rivers, with the coordinates -7.757560951020032, 6.349784100984212 serving as a reference point near the Nzo River.

    The **same data extraction methodology** is applied to all three crop types. The only variation is the **specific set of polygons** used for each crop, ensuring that the extracted data is inherently labeled by its source region, reflecting the distinct fields where each crop is grown within these broader areas.


## 3. Data Extraction Methodology 📈

The data extraction process is performed programmatically using the Google Earth Engine Python API, optimized with multiprocessing for efficiency.

### 3.1 Time Series Definition

* **Observation Period:** From '2020-01-01' to '2023-12-31', spanning multiple years to capture inter-annual variability and full growth cycles.
* **Temporal Resolution:** Data is aggregated weekly, with a `TIME_STEP_DAYS` of 7 days. This provides a dense time series critical for capturing the phenological (seasonal growth) patterns of different crops.

### 3.2 Sentinel-2 Bands Selected

A comprehensive set of 10 Sentinel-2 spectral bands, vital for vegetation analysis and crop discrimination, are extracted:

| Sentinel-2 Band | GEE Band Name | Custom Name  | Description                           |
| :-------------- | :------------ | :----------- | :------------------------------------ |
| B2              | `B2`          | `blue`       | Blue (0.490 µm)                       |
| B3              | `B3`          | `green`      | Green (0.560 µm)                      |
| B4              | `B4`          | `red`        | Red (0.665 µm)                        |
| B5              | `B5`          | `rededge1`   | Vegetation Red Edge (0.705 µm)        |
| B6              | `B6`          | `rededge2`   | Vegetation Red Edge (0.740 µm)        |
| B7              | `B7`          | `rededge3`   | Vegetation Red Edge (0.783 µm)        |
| B8              | `B8`          | `nir`        | Near Infrared (0.842 µm)              |
| B8A             | `B8A`         | `nir08`      | Near Infrared (0.865 µm)              |
| B11             | `B11`         | `swir16`     | Short-wave Infrared (1.610 µm)        |
| B12             | `B12`         | `swir22`     | Short-wave Infrared (2.190 µm)        |

These bands provide a rich spectral signature, covering visible light, red-edge (sensitive to chlorophyll content), Near-Infrared (NIR, indicative of biomass), and Short-wave Infrared (SWIR, related to water content and leaf structure).

### 3.3 Preprocessing: Cloud Masking and Scaling

All Sentinel-2 images undergo critical preprocessing steps:

* **Cloud Masking:** The `QA60` band is used to identify and mask out pixels affected by clouds and cirrus, ensuring that only clear-sky observations are used for analysis.
* **Reflectance Scaling:** Raw digital numbers are converted to **surface reflectance values** by dividing by 10000. This standardizes the data to a 0-1 range, making it comparable across different acquisition dates and locations.

### 3.4 Data Aggregation and Feature Extraction

For each polygon and each 7-day interval:

* **Image Collection Filtering:** Sentinel-2 images are filtered by date and spatial bounds of the polygon.
* **Median Composite:** A **median composite** is generated from all available clear-sky images within the 7-day window. The median is chosen for its robustness against residual noise or outliers.
* **Mean Reduction:** The **mean** spectral value for each selected band is calculated across all pixels within the polygon's geometry from the median composite. This provides a representative spectral signature for the entire polygon at a 10-meter spatial resolution.

### 3.5 Parallel Processing and Output Format

* **Efficient Processing:** The entire extraction workflow is parallelized using Python's `multiprocessing` module. Each polygon-date combination forms a distinct task, distributed across multiple CPU cores to significantly speed up data generation.
* **Structured Output:** The extracted mean spectral values, along with a `polygon_id` (linking back to the specific field) and the `date` of observation, are compiled into a pandas DataFrame. Any missing data from GEE is represented as `NaN`.
* **Labeled Dataset:** Each resulting CSV file (e.g., `S2_7day_Palm.csv`) implicitly carries the crop label corresponding to the source polygons used. These individual crop-specific datasets are then combined to form the complete training dataset for the multi-class crop classification task.

---

## 4. Dataset Fields 📊

The final training dataset (after combining individual crop CSVs) will contain the following key fields:

* **`polygon_id`**: A unique identifier for each specific agricultural field/polygon.
* **`date`**: The start date of the 7-day observation period (YYYY-MM-DD).
* **`blue`**, **`green`**, **`red`**, **`nir`**, **`swir16`**, **`swir22`**, **`rededge1`**, **`rededge2`**, **`rededge3`**, **`nir08`**: Mean surface reflectance values for the respective Sentinel-2 spectral bands.
* **`crop_label`**: The categorical label indicating the crop type ('Cocoa', 'Palm', 'Rubber'), derived from the source polygons used for data extraction.


### Second Dataset is the Côte d’Ivoire Byte-Sized Agriculture Challenge On zindi 

https://zindi.africa/competitions/cote-divoire-byte-sized-agriculture-challenge/data



