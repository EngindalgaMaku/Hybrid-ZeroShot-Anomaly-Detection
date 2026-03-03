# Hybrid Zero-Shot Anomaly Detection

> 🌐 **English** ✓ | **[Türkçe Versiyon (Turkish)](README.md)**

This project presents a hybrid approach for zero-shot anomaly detection in industrial/manufacturing data (e.g., MVTec AD dataset) using **CLIP** and **DINOv2** models. This work was prepared for an academic paper and includes the complete, executable codebase.

## Methods and Approaches

The project analyzes and compares three different anomaly detection approaches:

1. **Hybrid Approach**: 
   - Images are first screened using the **CLIP** model for image level filtering. This stage is very fast.
   - Only images flagged as suspicious are sent to the **DINOv2** model for detailed pixel level localization analysis.
   - This method achieves both speed and high detailed localization performance.
   
2. **CLIP-only Approach**:
   - Performs image level anomaly detection using only the CLIP model. Does not provide localization but runs quickly.
   
3. **DINO-only Approach**:
   - Performs pixel level anomaly detection and localization using only the DINOv2 model without any pre-filtering. Has the highest computational cost but provides the most detailed analysis.

## Dataset (MVTec AD)

This study uses **MVTec AD** (Anomaly Detection), a publicly available dataset that is widely used as a standard benchmark in industrial visual anomaly detection. 
To review and download the dataset, visit the official link:  
🔗 [MVTec AD Dataset Official Page](https://www.mvtec.com/company/research/datasets/mvtec-ad)

## Motivation and Research Scope

**Important Note:** The primary goal of this work is **not** to achieve the highest (State-of-the-Art / SOTA) anomaly detection scores in the existing literature. 
Our main motivation is to combine a model like CLIP, which operates very quickly at the image level but lacks localization capability, with a model like DINOv2, which provides highly detailed and accurate pixel level localization but has high computational cost, in a **hybrid** structure. 
Through this "gating" concept, we demonstrate that an **optimal balance between speed and accuracy (computational cost)** can be achieved by filtering thousands of normal products in industrial production lines within seconds with low hardware cost, and sending only suspicious (flagged) products to heavy anomaly mapping (DINOv2).

## Technologies Used and Reproducibility

- **OpenAI CLIP**: For extracting and filtering general image level anomaly features.
- **Meta DINOv2**: For pixel level mapping (patch level feature extraction), KNN indexing, and generating anomaly heatmaps.
- **Python Libraries**: `pandas`, `opencv-python`, `scikit-learn`, `ftfy`, `regex`, and PyTorch infrastructure for data management.
- **Reproducibility and Device Settings**: To ensure consistency and academic reproducibility of experiments, the random seed was fixed to `SEED = 42` (at Numpy, Random, and PyTorch CPU/GPU function levels). Experiments were conducted with reference to the following hardware configuration:
  - **Device used**: `cuda`
  - **GPU**: NVIDIA RTX PRO 6000 Blackwell Server Edition
  - **GPU Memory**: 101.97 GB

## Experimental Results

Average results from comparative experiments on all 15 categories in the MVTec AD dataset are summarized below. For the hybrid approach, the CLIP threshold quantile was selected as **0.93**:

| Method | Average Image AUROC | Average Pixel AUROC | Flagged Ratio |
|---|---|---|---|
| **CLIP-only** | **82.63%** (0.8263) | - (No Localization) | 100% (All examined) |
| **DINO-only** | 70.34% (0.7034) | **94.42%** (0.9442) | 100% (All processed heavily) |
| **Hybrid (CLIP+DINOv2)** | **82.63%** (0.8263) | 80.74% (0.8074) | **63.97%** (Only suspicious processed) |

**Result Analysis:** As shown in the table, the Hybrid method achieved significant computational savings by routing only **63.97%** of images to DINOv2. In addition to the naturally achieved image detection performance from CLIP (82.63%), the hybrid gating system added the detailed mapping power of DINOv2, obtaining a quite strong end-to-end (E2E) pixel level localization score of **80.74%**. While this represents a trade-off compared to the pure 94% localization of the DINO-only method, it is highly optimal for industrial applications considering the massive 36% hardware/speed savings achieved. All these output details are available in the `results/` folder within the repository.

### Threshold Selection (Ablation Study)
An "Ablation Study" (90%, 93%, 95%, 97%, 99% quantile) was conducted to determine exactly which CLIP Score (Quantile) the Hybrid structure should use to route images to DINOv2. The following table clearly shows how the optimal balance (`0.93 - 93%` Quantile) was selected:

| Selected Quantile (Threshold Ratio) | Average Image AUROC | Average Conditional Pixel AUROC | Average E2E Pixel AUROC | Routing Rate to DINO (Flag) |
|---|---|---|---|---|
| 0.90 | 0.8263 | 0.9393 | **0.8204** | 68.43% |
| **0.93 (Selected)** | **0.8263** | **0.9395** | 0.8074 | **63.97%** |
| 0.95 | 0.8263 | 0.9378 | 0.7989 | 60.98% |
| 0.97 | 0.8263 | 0.9381 | 0.7891 | 58.60% |
| 0.99 | 0.8263 | 0.9416 | 0.7643 | 52.40% |

*(With the selected `0.93` threshold, the filtering rate was advantageously reduced to approximately 64%, while no significant weakness was experienced in end-to-end localization (E2E Pixel AUROC) performance.)*

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
- Calculation of **AUROC** metrics (image level and pixel level) separately for each category and tested method.
- Reporting of comparative and detailed metrics in **Excel** and **CSV** formats.
- Generation of 3-panel qualitative visualizations suitable for publication:
  - Panel 1: Original image, CLIP score, and threshold status.
  - Panel 2: Ground truth mask (MVTec AD references).
  - Panel 3: DINOv2 heatmap (localization result).

## Usage

To get started, `Hybrid-ZeroShot-Anomaly-Detection.ipynb` in the repository can be run in Google Colab or Jupyter environment. Dataset directory paths are configured to be pulled from Google Drive within Colab. Before running, it is recommended to adjust directory structures including `MVTEC_ROOT` according to your own Google Drive or local system.