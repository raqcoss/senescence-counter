# Senescence Counter

A computer vision tool for automated senescence quantification in microscopy images of $\beta$-galactosidase ($\beta$-gal) stained *in vitro* cell cultures.

## Overview

Senescent cells can be differentiated by the enzymatic activity of $\beta$-gal, which is observed via the hydrolysis of its substrate, $\beta$-galactosides, into simpler monosaccharides. To assess senescence, this pipeline calculates the ratio of $\beta$-gal-positive (blue-stained) cells against the total cell population. Total cell counts are quantified using DAPI nuclear staining in corresponding fields of view.

---

## Processing Pipeline

Given a pair of images per field of view (**DAPI** and **$\beta$-gal**), the pipeline executes the following steps:

[ DAPI Image ]      -->  CLAHE + Morphological Filter  -->  Otsu Thresholding  -->  Nuclei Centroids

[ β-gal Image ]     -->  HSV Color Space + Blue Mask   -->  Morphological Clean --> Spatial Matching

Senescence Ratio (%) <-- number of nuclei inside blue $\beta$-gal stained zones / total nuclei

### 1. DAPI Preprocessing & Nuclei Segmentation
* **Contrast Enhancement:** CLAHE on the L-channel in LAB color space (`clipLimit = 4.0`, `tileGridSize = (9, 9)`).
* **Noise Reduction & Clean-up:** Optional sharpening ($3 \times 3$ kernel) followed by morphological opening (top-hat-like filtering with a $9 \times 9$ structuring element).
* **Binarization:** Otsu thresholding on grayscale followed by small-structure removal to isolate single nuclei.

<img src="data/visualizations/dapi_count_example.png" alt="results for counting DAPi stained nuclei" style="height: 300px; width:500px;"/>


### 2. Senescence ($\beta$-gal) Masking
* **Color Space Transformation:** Conversion to HSV color space.
* **Hue & Saturation Filtering:** Blue segmentation using hue windowing ($H \in [40, 140]$, $S \ge 10$, $V \ge 15$).
* **Relative Blue-Dominance Masking:** Intersected with relative blue dominance:
  
  $D_{\mathrm{rel}} = \frac{\max(0, B - \frac{R + G}{2})}{V + \epsilon} \cdot 255 >$ `blue_dom_thresh`

* **Morphological Refinement:** Sequential morphological opening ($2\times$) and closing ($1\times$) using a $9 \times 9$ elliptical kernel.

### 3. Cell Matching & Counting
* A nucleus is classified as **senescent** if its centroid lies within a segmented blue region.
* If no centroid lies directly inside, the single nearest nucleus within `near_radius` is counted.
* **Agglomerate Handling:** Blue regions strongly overlapped by agglomerates can be excluded using `ignore_aggl = True`. Otherwise, each agglomerated region contributes one senescent cell and increments the total nuclei count by one.

### 4. Quantification
   $\text{Senescence\%} = \frac{\text{num senescent cells}}{\text{num total (single) nuclei}} \times 100$

---

## Repository Structure

```text
senescence-counter/
├── data/                  # Sample/input images (DAPI and beta-gal)
├── sene_count.ipynb       # Main processing notebook
├── results/               # Output masks, overlays, and exported CSVs
├── README.md              # Project documentation
└── requirements.txt       # Python dependencies
```

## Prerequisites

Ensure you have Python 3.8+ installed along with the required libraries:
Bash

```bash
pip install numpy opencv-python matplotlib jupyterlab
```

## Running the Notebook

Clone this repository:

```bash
git clone [https://github.com/your-username/senescence-counter.git](https://github.com/your-username/senescence-counter.git)
    cd senescence-counter
```

## Launch Jupyter Lab / Notebook:

```bash
jupyter lab notebooks/pipeline.ipynb
```

Place your image pairs into the `data/` directory following the naming pattern expected by the notebook script.

## Contribuitors

- Raquel Cossío Ramírez - *code development*

- Maria Trejo - *model evaluation*

- Lorelei Xiadani Ayala Guerrero (Instituto de Fisiología Celular at UNAM)- *provided dataset of original pictures and human validation results*
