# senescence-counter
Computer vision tool for senecence quantification in microscopy images of beta-Gal stained *in vitro* cultures

Senecent cells can be differentiated by the enzimatic activity of $\beta$-gal which can be observed by the rupture (hydrolisis) of its substrate, $\beta$-galactosides into simpler sugars (monosaccharides). To asess senescence, we calcule the number of cells marked with blue aganist the total number of cells in a sample. Total cells are easily quantifiable in confocal microscopy with DAPi immunostaining.

## Steps:

Given a pair of images per field of view (DAPI and $\beta$-gal), our pipeline performs:

1. **Preprocessing**: DAPI images are preprocessed with CLAHE on the L channel in LAB (`clipLimit = 4.0`, `tileGridSize` = 9 x 9), then optional sharpening (3 x 3 kernel), and morphological open (top-hat-like clean-up with 9 x 9 structuring element) is performed. Afterwards we apply Otsu thresholding on grayscale and small-structure removal.
2. **Senescence mask**: $\beta$-gal images are converted into HSV color space, where blue color is segmented according to a preset hue windowing (H $\in$ [40,140], S $\ge$ 10, V $\ge$ 15) intersected with a relative blue-dominance mask $D_\mathrm{rel} = \frac{\max(0, B-\tfrac{R+G}{2})}{V+\epsilon}\cdot255$ > `blue_dom_thresh`, then morphological open (2x) and close (1x) using $9\times9$ elliptical kernel is applied.
3. **Counting senescent cells**: A nucleus is senescent if its centroid lies within a blue region; if none lies inside, count the single nearest nucleus within `near_radius`. Blue regions strongly overlapped by agglomerates are excluded when `ignore_aggl` is True; otherwise, each such region counts as one senescent cell and increments total nuclei by one.
4. **Senescence percentage**:

   $\text{Senescence\%} = \frac{\text{num senescent cells}}{\text{num total (single) nuclei}} \times 100$

## Contribuitors

- Raquel Cossío Ramírez - *code development*

- Maria Trejo - *model evaluation*

- Lorelei Xiadani Ayala Guerrero (Instituto de Fisiología Celular at UNAM)- *provided dataset of original pictures and human validation results*
