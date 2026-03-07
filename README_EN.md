# Hybrid Zero-Shot Anomaly Detection

> 🌐 **English** ✓ | **[Türkçe Versiyon (Turkish)](README.md)**

This project presents a hybrid approach for zero-shot anomaly detection in industrial/manufacturing data (e.g., MVTec AD dataset) using **CLIP** and **DINOv2** models. This work was prepared for an academic paper and includes the complete, executable codebase.

## Methods and Approaches

The project analyzes and compares three different anomaly detection approaches:

1. **Hybrid Approach**: 
   - Images are first screened using the **CLIP** model for image-level filtering. This stage is very fast.
   - Only images flagged as suspicious are sent to the **DINOv2** model for detailed pixel-level localization analysis.
   - This method achieves both speed and high detailed localization performance.
   
2. **CLIP-only Approach**:
   - Performs image-level anomaly detection using only the CLIP model. Does not provide localization but runs quickly.
   
3. **DINO-Max Approach**:
   - Performs pixel-level anomaly detection and localization using only the DINOv2 model without any pre-filtering.
   - **Image-level scoring** is calculated using **max aggregation** (maximum value from patch anomaly map), which is the literature standard.
   - Has the highest computational cost but provides the most detailed analysis.

## Dataset (MVTec AD)

This study uses **MVTec AD** (Anomaly Detection), a publicly available dataset that is widely used as a standard benchmark in industrial visual anomaly detection. 
To review and download the dataset, visit the official link:  
🔗 [MVTec AD Dataset Official Page](https://www.mvtec.com/company/research/datasets/mvtec-ad)

## Motivation and Research Scope

**Important Note:** The primary goal of this work is **not** to achieve the highest (State-of-the-Art / SOTA) anomaly detection scores in the existing literature. 
Our main motivation is to combine a model like CLIP, which operates very quickly at the image-level but lacks localization capability, with a model like DINOv2, which provides highly detailed and accurate pixel-level localization but has high computational cost, in a **hybrid** structure. 
Through this "gating" concept, we demonstrate that an **optimal balance between speed and accuracy (computational cost)** can be achieved by filtering thousands of normal products in industrial production lines within seconds with low hardware cost, and sending only suspicious (flagged) products to heavy anomaly mapping (DINOv2).

## Technical Details

### DINOv2 Image-Level Scoring
This study uses the literature standard **max aggregation** method. DINOv2 generates a patch-level anomaly map for each image and calculates the image-level score as the **maximum value** from this map:

```python
# Literature standard
image_score = np.max(anomaly_map)
```

This approach is widely used in WinCLIP, PatchCore, and other patch-based anomaly detection methods.

### Distance Metric
**Cosine distance** metric is used for k-NN (k-Nearest Neighbors) operations:

```python
nn = NearestNeighbors(n_neighbors=k, algorithm="auto", metric="cosine")
```

Features are L2 normalized before being compared with cosine distance, which is a standard practice in the literature.

## Technologies Used and Reproducibility

- **OpenAI CLIP**: For extracting and filtering general image-level anomaly features.
- **Meta DINOv2**: For pixel-level mapping (patch-level feature extraction), KNN indexing, and generating anomaly heatmaps.
- **Python Libraries**: `pandas`, `opencv-python`, `scikit-learn`, `ftfy`, `regex`, and PyTorch infrastructure for data management.
- **Reproducibility and Device Settings**: To ensure consistency and academic reproducibility of experiments, the random seed was fixed to `SEED = 42` (at Numpy, Random, and PyTorch CPU/GPU function levels). Experiments were conducted with reference to the following hardware configuration:
  - **Device used**: `cuda`
  - **GPU**: NVIDIA RTX 4060
  - **GPU Memory**: 8 GB
  - **RAM**: 16 GB

## Experimental Results

Average results from comparative experiments on all 15 categories in the MVTec AD dataset are summarized below. For the hybrid approach, the CLIP threshold quantile was selected as **0.93**:

| Method | Image AUROC | Pixel AUROC (E2E) | Pixel AUROC (Cond) | Flag Rate | Total | Per Image | FPS |
|---|---|---|---|---|---|---|---|
| **CLIP-only** | **82.58%** | - | - | - | 113.1s | 66ms | 15.3 |
| **DINO-Max** | **85.55%** | **95.61%** | - | 100% | 812.7s | 471ms | 2.1 |
| **Hybrid** | 82.58% | 81.31% | **95.22%** ⭐ | **50.5%** | **532.6s** ✅ | 309ms | 3.2 |

**Note:** Measured on 1725 test images (15 categories total). Hybrid method is **34.5% faster** than DINO-Max.

### Result Analysis

**✅ Key Findings:**

1. **Speed Improvement:** Hybrid method is **34.5% faster** than DINO-Max (532.6s vs 812.7s)
   - This is achieved through an average **50.5% flag rate**
   - Only suspicious images are sent to DINOv2

2. **Localization Quality:** When DINO runs (flagged images), localization quality is **almost identical**
   - Conditional Pixel AUROC: **95.22%** (Hybrid) vs 95.61% (DINO-Max)
   - Difference is only **0.39 points** - statistically negligible

3. **Trade-off:** End-to-End Pixel AUROC is lower (81.31%) because:
   - CLIP misses some anomalies (false negatives)
   - These images are not sent to DINOv2, so no localization is performed
   - However, this is an acceptable trade-off for **speed gains**

4. **Image-Level Performance:**
   - CLIP and Hybrid are the same (82.58%) - because Hybrid uses CLIP scores
   - DINO-Max is slightly better (85.55%) but the difference is small (+2.97 points)

### Performance by Category

**Best Categories (Hybrid successful):**
- Bottle: Image 99.0%, Pixel 98.3%, Flag 73.5%
- Leather: Image 99.5%, Pixel 98.9%, Flag 79.0%
- Wood: Image 99.8%, Pixel 94.9%, Flag 77.2%
- Tile: Image 98.9%, Pixel 94.7%, Flag 76.9%

**Challenging Categories (CLIP weak):**
- Cable: Image 58.9% (DINO-Max: 87.5%), Flag only 32%
- Capsule: Image 67.9% (DINO-Max: 77.4%), Flag only 24%
- Screw: Image 53.7% (DINO-Max: 64.2%), Flag only 19%

**Conclusion:** Hybrid method works excellently in categories where CLIP is strong. For challenging categories (cable, capsule), DINO-Max may be preferred.

### Threshold Selection (Ablation Study)
An "Ablation Study" (90%, 93%, 95%, 97%, 99% quantile) was conducted to determine exactly which CLIP Score (Quantile) the Hybrid structure should use to route images to DINOv2. The following table clearly shows how the optimal balance (`0.93 - 93%` Quantile) was selected:

| Selected Quantile (Threshold Ratio) | Average Image AUROC | Average Conditional Pixel AUROC | Average E2E Pixel AUROC | Routing Rate to DINO (Flag) |
|---|---|---|---|---|
| 0.90 | 0.8258 | 0.9526 | **0.8273** | 55.2% |
| **0.93 (Selected)** | **0.8258** | **0.9522** | 0.8131 | **50.47%** ⭐ |
| 0.95 | 0.8258 | 0.9512 | 0.8037 | 47.7% |
| 0.97 | 0.8258 | 0.9541 | 0.7879 | 44.7% |
| 0.99 | 0.8258 | 0.9538 | 0.7674 | 39.6% |

**Why was 0.93 selected?**
- ✅ Flag rate 50.47% → Medium-level savings
- ✅ Conditional Pixel AUROC 95.22% → Excellent localization quality
- ✅ E2E Pixel AUROC 81.31% → Reasonable trade-off
- ✅ Balance point: Optimal between speed and quality

## Visual Analysis (Qualitative Results)

The following are 3-panel qualitative visualizations produced to demonstrate how the hybrid anomaly detection pipeline works. 
**[Panel 1]** Image and CLIP Threshold Status | **[Panel 2]** Ground Truth Error Mask | **[Panel 3]** DINOv2 Heatmap Localization

![Bottle Anomaly Example](results/qual_panels_balanced/bottle/bottle_00_flagT_clip0.028.png)
![Capsule Anomaly Example](results/qual_panels_balanced/capsule/capsule_00_flagT_clip0.059.png)
![Carpet Anomaly Example](results/qual_panels_balanced/carpet/carpet_00_flagT_clip0.046.png)
![Grid Anomaly Example](results/qual_panels_balanced/grid/grid_00_flagT_clip0.035.png)

*(More visual examples and details can be found in the `results/qual_panels_balanced/` directory.)*

## Outputs and Analysis

The system achieves the following analysis and evaluation objectives:
- Calculation of **AUROC** metrics (image-level and pixel-level) separately for each category and tested method.
- Reporting of comparative and detailed metrics in **Excel** and **CSV** formats.
- Generation of 3-panel qualitative visualizations suitable for publication:
  - Panel 1: Original image, CLIP score, and threshold status.
  - Panel 2: Ground truth mask (MVTec AD references).
  - Panel 3: DINOv2 heatmap (localization result).

## Complexity Analysis (Computational Complexity)

### Theoretical Big-O Analysis

**Parameters:**
- N: Number of test images
- P: Number of patches per image (~256-1024)
- B: Patch bank size (~10,000)
- k: Number of k-NN neighbors (k=5)
- α: Flag rate (0 ≤ α ≤ 1)

**Methods:**

1. **CLIP-only:** O(N)
   - Constant processing per image
   
2. **DINO-Max:** O(N · P · log(B))
   - P patches × k-NN search per image
   
3. **Hybrid:** O(N) + O(N · α · P · log(B))
   - CLIP gating: O(N)
   - DINOv2 (only flagged): O(N · α · P · log(B))
   - α ≈ 0.49 → **~50% savings**

### Empirical Runtime

| Method | Per Category | Total (15 categories) | Per Image | FPS | Savings |
|---|---|---|---|---|---|
| CLIP-only | 7.5s | 113.1s | 66ms | 15.3 | - |
| DINO-Max | 54.2s | 812.7s | 471ms | 2.1 | - |
| **Hybrid** | **35.5s** | **532.6s** | **309ms** | **3.2** | **-34.5%** ⚡ |

## Usage

To get started, `Hybrid-ZeroShot-Anomaly-Detection.ipynb` in the repository can be run in Google Colab or Jupyter environment. 

**Automatic Environment Detection:** The notebook works in both local and Colab environments:
- In Colab: Automatic Drive mount + Colab paths
- Locally: Windows/Linux/Mac paths

Before running, it is recommended to adjust the `MVTEC_ROOT` directory path according to your own system.

## License and References

This project was prepared for academic research purposes. Models used:
- **CLIP:** OpenAI (https://github.com/openai/CLIP)
- **DINOv2:** Meta AI (https://github.com/facebookresearch/dinov2)
- **MVTec AD:** MVTec Software GmbH (https://www.mvtec.com/company/research/datasets/mvtec-ad)