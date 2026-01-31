
---

# 🏨 Persona-Aware Hotel Discovery

**Booking.com Data Collection, Enrichment & Modeling Project**

---

## 📌 Project Overview

This project builds a **persona-aware hotel recommendation system** using Booking.com listings.

Instead of relying on manual labels, we:

1. Clean and standardize raw Booking.com data.
2. Enrich listings with **neighborhood context** using OpenStreetMap (Overpass API).
3. Generate **weak labels** (Family / Remote Worker / Tourist) from reviews and descriptions.
4. Validate those labels by training **student ML models** on structured data only.
5. Produce **persona probability scores** for all listings.
6. Expose results via **interactive Databricks UI notebooks**.

### Supported Personas

* 👨‍👩‍👧 **Family**
* 💻 **Remote Worker**
* 🧳 **Tourist**

---

## 🧱 Project Structure & Execution Order

⚠️ **Important:**
Notebooks are designed to be run **top-to-bottom** and **in order**, unless stated otherwise.

---

### **Stage 0 — Data Exploration**

📓 `01_Booking_EDA.ipynb`

* Validates dataset granularity (`hotel_id` uniqueness)
* Analyzes missingness (including nested arrays / structs)
* Confirms feasibility of:

  * Weak supervision
  * Spatial enrichment

**Output:**
*No files written (analysis only)*

---

### **Stage 1 — Booking Dataset Cleaning**

📓 `02_Booking_Cleaning.ipynb`

* Deterministic cleaning (no imputation)
* Normalizes strings, arrays, and nested structs
* Extracts latitude / longitude
* Canonicalizes room capacity
* Enforces **1 row = 1 listing**

**Output:**
`dbfs:/tmp/booking_clean/booking_clean.parquet`

---

### **Stage 3 — Grid Selection + OSM Enrichment**

📓 `03_Grid_Selection_Overpass_Enrichment.ipynb`

* Builds 50km coarse regions and 1km fine grids
* Selects dense grids for scraping
* Scrapes POIs via **Overpass API**
* Extracts:

  * POI counts
  * Minimum distances per category
* Fully **resumable scraping**

**Output:**
`dbfs:/tmp/booking_grid_enrichment_final`

---

### **Stage 4A — Grid-Level Feature Engineering**

📓 `04_Feature_Engineering_Grid.ipynb`

* Recomputes `grid_id` (same logic as Stage 3)
* Joins listings with grid enrichment
* Engineers ML-friendly features:

  * log(counts)
  * proximity flags
  * inverse distances

**Outputs:**

* `dbfs:/tmp/booking_stage4/booking_listing_features_full`
* `dbfs:/tmp/booking_stage4/booking_listing_features_enriched`
* `dbfs:/tmp/booking_stage4/scraped_enrichment_features_ml`

---

### **Stage 4B — Booking Core Feature Engineering**

📓 `04_Feature_Engineering_Booking_Core.ipynb`

* Extracts structured Booking features:

  * Amenities & facilities
  * Capacity & room diversity
  * House rules (pets, quiet hours, age restriction)
  * Landmark proximity

**Output:**
`dbfs:/tmp/booking_stage4/booking_features_complete_ml`

---

### **Stage 4C — Final Feature Assembly (No Labels)**

📓 `04_Final_Feature_Assembly.ipynb`

* Joins Booking core features with scraped enrichment
* Ensures:

  * One row per `hotel_id`
  * No column collisions

**Output:**
`dbfs:/tmp/booking_stage4/final_features_assembled_no_labels`

---

### **Stage 4D — Teacher Label Generation (Weak Supervision)**

📓 `04_Teacher_Labeling.ipynb`

* Rule-based persona scoring over:

  * Reviews
  * Descriptions
* Handles:

  * Negations
  * Negative evidence penalties
* Produces **multi-label persona assignments**

**Output:**
`dbfs:/tmp/booking_stage4/teacher_labels_multilabel_desc_reviews_v2`

---

### **Stage 5 — Student Model Training & Evaluation**

📓 `05_Model_Training_XGBoost.ipynb`

* Trains XGBoost classifiers (one per persona)
* Compares:

  * Base features
  * Base + enrichment features
* Metrics:

  * ROC-AUC
  * PR-AUC
  * F1 (default and optimized threshold)

**Outputs:**

* `dbfs:/tmp/booking_stage5/models_with_enrichment_v1`
* `dbfs:/tmp/booking_stage5/models_without_enrichment_v1`

---

### **Stage 6 — Persona Probability Generation**

📓 `07_Generate_Persona_Probabilities.ipynb`

* Loads trained models
* Generates persona probabilities for all listings
* Uses enriched models when available, otherwise base models

**Outputs:**

* `dbfs:/tmp/booking_stage5/predictions_v1/pred_base_all`
* `dbfs:/tmp/booking_stage5/predictions_v1/pred_enriched_only`
* `dbfs:/tmp/booking_stage5/predictions_v1/pred_combined_best`

---

## 🖥️ User Interfaces (Two Options)

### **UI Option 1 — Full DBFS-Based UI**

📓 `08_Persona_Aware_UI_DBFS.ipynb`

* Reads directly from DBFS prediction outputs
* Requires **running the full pipeline first**
* Intended for full technical evaluation

---

### **UI Option 2 — Lightweight Sample UI (Recommended)**

📓 `08_Persona_Aware_UI_Sample.ipynb`

✅ **Fastest way to see results**

* Uses a **pre-generated sample CSV**
* Does **NOT** require running the full project
* Requires only:

  1. Pasting the **SAS token** provided by course staff
  2. Clicking **Run All**

**Input file (course storage):**
`abfss://submissions@lab94290.dfs.core.windows.net/<group_folder>/ui_sample_hotels.csv`

This UI allows immediate exploration of:

* Persona selection
* Country / city filtering
* Ranked hotel recommendations

---

## 🔐 Authentication & Security

* **No SAS tokens are stored in Git**
* Token must be pasted manually in UI notebooks
* All paths are reproducible on the course environment

---

## ✅ How to Run (Recommended)

**Fastest evaluation path:**

1. Open **UI Option 2 — Sample UI**
2. Paste SAS token
3. Click **Run All**
4. Explore recommendations interactively

**Full pipeline evaluation:**

1. Run notebooks in order (Stage 0 → Stage 7)
2. Open **UI Option 1 — DBFS UI**

---

