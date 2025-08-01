# Amini GeoFM Decoding the Field Challenge

## Overview
The Amini GeoFM Decoding the Field Challenge leverages Geospatial Foundational Models (GeoFMs) to classify cocoa, palm, and rubber crops using Sentinel-2 multispectral imagery time-series data. The goal is to automate crop type classification, reducing reliance on labor-intensive field surveys and enabling faster agricultural decision-making for yield estimation and policy support.

## Objective
Develop a classifier using GeoFM embeddings from PRESTO and/or Amini Earth FM to predict crop types (Cocoa, Palm, Rubber) with high accuracy on a test set validated by ground survey data.

## Project Structure
- **Data_extraction_coco_palm_rubber_crops.ipynb**: Extracts Sentinel-2 data and GeoFM embeddings via Google Earth Engine.
- **amini-geofm-crop-classification-first-notebook.ipynb**: Trains LightGBM classifier and generates predictions.
- **lstm-amini-field-challenge-final-notebook.ipynb**: Trains LSTM model and applies ensemble strategy.
- **amini-analysis.ipynb**: Analyzes data and model performance.
- **Training_Data_Crop_Classification.csv**: Training dataset.
- **requirements.txt**: Python dependencies.

## Methodology
1. **Data Extraction and Preprocessing**
   - Extract Sentinel-2 multispectral data and GeoFM embeddings using Google Earth Engine.
   - Generate `Training_Data_Crop_Classification.csv`.

2. **Model Development**
   - **LightGBM Classifier**: Trains on extracted features, captures non-linear relationships, and generates predictions for Cocoa.
   - **LSTM Classifier**: Processes time-series data to capture temporal patterns, used for Palm and Rubber predictions.

3. **Ensemble and Post-processing**
   - Combines LightGBM (Cocoa) and LSTM (Palm, Rubber) predictions.
   - Applies outlier detection (IQR method) and replaces outliers with median values.
   - Clips probabilities to ensure valid ranges.

## How to Run
### Prerequisites
- Python 3.8+
- Google Earth Engine account
- Kaggle or local setup with sufficient resources

### Steps
1. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

2. **Data Extraction**
   ```bash
   jupyter notebook Data_extraction_coco_palm_rubber_crops.ipynb
   ```
   - Extracts Sentinel-2 data and generates training dataset.

3. **LightGBM Training**
   ```bash
   jupyter notebook amini-geofm-crop-classification-first-notebook.ipynb
   ```
   - Trains LightGBM model, saves predictions as `Amini-geofm-sub (3).csv`.

4. **LSTM Training and Ensemble**
   ```bash
   jupyter notebook lstm-amini-field-challenge-final-notebook.ipynb
   ```
   - Trains LSTM model, combines predictions, applies outlier handling, and saves final submission as `weighted_sub.csv`.

5. **Analysis (Optional)**
   ```bash
   jupyter notebook amini-analysis.ipynb
   ```
   - Visualizes data and model performance.

## Key Features
- **Ensemble Strategy**: Combines LightGBM (Cocoa) and LSTM (Palm, Rubber) for optimized predictions.
- **Outlier Handling**: Uses IQR to detect and replace outliers with median values.
- **Probability Clipping**: Ensures stable predictions by clipping probabilities.
- **Optimizations**:
  - Batch processing for memory efficiency.
  - Feature engineering with spectral and temporal data.
  - Cross-validation to prevent overfitting.

## Performance Tips
- Use GPU acceleration for faster training.
- Cache data for repeated experiments.
- Monitor memory usage during large-scale processing.

### Best submission info:
   - ID: Ufib53cr
   - Filename: weighted_sub.csv
   - Comment: —
   - Submitted: 12 July 00:01