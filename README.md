# Multimodal Remote Sensing Semantic Segmentation with AnySat

Multimodal land-cover semantic segmentation on the FLAIR-HUB benchmark using the AnySat Earth observation foundation model.

This project was developed as my MSc thesis at the Budapest University of Technology and Economics (BME). It investigates parameter-efficient adaptation of AnySat, multimodal fusion strategies, spatial post-processing, and interpretable modality/channel importance analysis for remote sensing segmentation.

## Highlights

- Adapted the pretrained **AnySat** foundation model to a 15-class land-cover segmentation task.
- Constructed a multimodal FLAIR-HUB subset containing **35,850 samples** from **10 spatially separated regions**.
- Integrated aerial imagery, elevation data, SPOT imagery, Sentinel-1 SAR, and Sentinel-2 multispectral time series.
- Implemented a **two-stage LoRA fine-tuning strategy** for parameter-efficient adaptation.
- Compared **native sequence-level fusion**, **strict early fusion**, and **late token-gated fusion**.
- Evaluated **DenseCRF** as an independent spatial refinement module.
- Analyzed sensor contribution using both **ablation-based** and **gate-based** modality/channel importance estimation.

## Task Overview

Multimodal Earth observation data provide complementary information at different spatial, spectral, and temporal resolutions.

High-resolution aerial imagery provides detailed spatial structure, while satellite optical data contribute spectral information, SAR observations provide complementary sensing characteristics, and elevation data help distinguish objects with similar appearance but different physical structure.

The goal of this project is to study how these heterogeneous sources can be effectively integrated into a unified semantic segmentation framework.

The experimental pipeline focuses on three questions:

1. How effectively can AnySat be adapted to FLAIR-HUB using parameter-efficient fine-tuning?
2. At which stage should heterogeneous modalities be fused?
3. Which modalities and channels contribute most to the final segmentation performance?

## Dataset

The experiments use a spatially distributed subset of the **FLAIR-HUB** multimodal remote sensing benchmark.

### Experimental subset

| Property | Value |
| --- | --- |
| Spatial regions | 10 |
| Samples | 35,850 |
| Approximate coverage | 375.91 km² |
| Evaluation classes | 15 |
| Split strategy | Spatial block split |

Spatial block splitting is used to reduce geographic leakage between training, validation, and test data.

### Input modalities

| Modality | Representation | Main information |
| --- | --- | --- |
| Aerial-FLAIR | RGBI + nDSM, 512 × 512 | High-resolution spatial structure and elevation |
| SPOT | RGB, 100 × 100 | Medium-resolution optical context |
| Sentinel-1 DESC | VV, VH, ratio | SAR information |
| Sentinel-1 ASC | VV, VH | Complementary SAR viewing geometry |
| Sentinel-2 | 10 spectral bands, temporal sequence | Multispectral and temporal information |

For the aerial modality,

**nDSM = DSM − DTM**

is used to provide relative elevation information.

Sentinel-1 ascending and descending acquisitions are kept separately, while Sentinel-2 observations are organized temporally after cloud filtering and temporal aggregation.

## Preprocessing

The multimodal preprocessing pipeline includes:

- Per-modality z-score normalization
- Spatial alignment across heterogeneous resolutions
- Synchronized spatial augmentation
- Sentinel-2 cloud filtering
- Temporal organization of satellite observations
- Label remapping to 15 valid evaluation classes
- Class weighting for long-tailed land-cover distributions

The selected dataset contains substantial class imbalance, with rare classes such as greenhouse and swimming pool occupying only a small fraction of the samples. Weighted cross-entropy is therefore used during training.

## Model

The system is built around the pretrained **AnySat** multimodal Earth observation foundation model.

Instead of fully fine-tuning the entire backbone, the model is adapted using a two-stage strategy.

### Stage 1 — Segmentation Head Adaptation

The pretrained backbone is initially frozen while the segmentation head is trained on the FLAIR-HUB target task.

### Stage 2 — LoRA Fine-Tuning

Low-Rank Adaptation (LoRA) is introduced into selected transformer blocks together with the segmentation head and modality-specific components.

This allows the pretrained representation to adapt to the target domain while updating only a relatively small number of parameters.

## Multimodal Fusion

Three fusion strategies are evaluated under the same experimental protocol.

### 1. Native AnySat Fusion

The original AnySat design preserves modality-specific projection before combining multimodal tokens in the transformer representation.

This serves as the main baseline.

### 2. Strict Early Fusion

All modalities are aligned and compressed into a shared representation before the common encoder.

This strategy tests whether very early cross-modal interaction benefits the segmentation task.

### 3. Late Token-Gated Fusion

Each modality is first encoded separately through the shared backbone.

A learnable token-level gating mechanism then combines the modality representations before dense segmentation.

This design preserves modality-specific information for longer and allows the model to selectively weight different sources.

## Main Results

| Method | mIoU | Mean F1 |
| --- | ---: | ---: |
| Native AnySat baseline | **0.5964** | **0.7301** |
| Late fusion | 0.5947 | 0.7282 |
| Strict early fusion | 0.5218 | 0.6654 |

The late-fusion model performs very close to the native AnySat baseline, with only a small decrease in mIoU.

In contrast, strict early fusion produces a considerably larger performance drop. This suggests that aggressively compressing heterogeneous modalities before sufficient modality-specific representation learning can discard useful information.

The results support preserving modality-specific representations until a later stage of the network.

## DenseCRF Refinement

Dense Conditional Random Fields were evaluated as a model-independent post-processing step.

The segmentation softmax probabilities are used as unary potentials, while Gaussian and bilateral pairwise terms incorporate spatial and appearance information from high-resolution aerial data.

| Method | mIoU | Mean F1 |
| --- | ---: | ---: |
| Baseline | 0.5964 | 0.7301 |
| Baseline + DenseCRF | **0.6028** | **0.7349** |

DenseCRF provides a modest improvement in overall segmentation performance and helps refine local spatial consistency and object boundaries.

## Modality and Channel Importance

Interpretability is studied from two complementary perspectives.

### Ablation-Based Importance

Individual modalities or channels are removed from the input and the resulting performance degradation is measured.

A larger performance drop indicates stronger dependence on the removed input.

### Gate-Based Importance

Learnable gates estimate how strongly the model retains or suppresses different input modalities and channels.

Combining the two approaches provides both:

- **performance sensitivity**, measured through ablation;
- **learned model preference**, measured through gating.

The experiments show that high-resolution aerial information forms the dominant spatial representation, while Sentinel-2 provides important complementary spectral information, particularly through near-infrared, red-edge, and short-wave infrared bands.

## Training Configuration

The main experiments use:

- PyTorch
- AnySat pretrained backbone
- AdamW optimizer
- Two-stage fine-tuning
- LoRA adaptation
- Weighted cross-entropy
- OneCycle learning-rate scheduling

The experimental pipeline was designed so that different fusion strategies could be compared under consistent preprocessing, training, and evaluation settings.

## Evaluation

Model performance is evaluated using:

- Mean Intersection over Union (**mIoU**)
- Per-class IoU
- Mean F1 score
- Per-class F1
- Qualitative segmentation maps
- Modality ablation
- Channel ablation
- Learned gating weights

Both aggregate and class-level results are used because the FLAIR-HUB subset contains strongly imbalanced land-cover categories.

## Project Structure

```text
anysat-multimodal-segmentation/
├── configs/                 # Experiment configurations
├── data/                    # Dataset preparation utilities
├── models/                  # AnySat adaptation and fusion modules
├── training/                # Training and fine-tuning pipeline
├── evaluation/              # Segmentation metrics and evaluation
├── analysis/                # Modality/channel importance analysis
├── crf/                     # DenseCRF refinement
├── scripts/                 # Experiment scripts
├── assets/                  # Figures and qualitative results
├── requirements.txt
└── README.md
```

## Research Context

This work was conducted as an MSc thesis at the **Budapest University of Technology and Economics (BME)**.

The thesis focuses on interpretable multimodal remote sensing segmentation using Earth observation foundation models.

A journal manuscript based on the project is currently being prepared.

## Future Work

Potential extensions include:

- More selective cross-modal fusion mechanisms
- Cross-attention-based fusion
- Improved learning for rare land-cover classes
- Learnable boundary-refinement modules
- Larger-scale FLAIR-HUB evaluation
- More detailed modality/channel interpretability analysis
- Efficiency analysis for multimodal inference

## Acknowledgements

This work builds upon the AnySat foundation model and the FLAIR-HUB multimodal remote sensing benchmark.

The project was supervised at BME with research support from HUN-REN SZTAKI.
