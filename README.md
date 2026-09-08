# Multimodal Remote Sensing Segmentation with AnySat

Multimodal land-cover semantic segmentation on FLAIR-HUB using the AnySat Earth observation foundation model.

This project was developed as my MSc thesis at the Budapest University of Technology and Economics (BME), focusing on parameter-efficient fine-tuning, multimodal fusion, and modality importance analysis.

## Overview

The project adapts the pretrained **AnySat** foundation model to a 15-class land-cover segmentation task using multimodal remote sensing data from **FLAIR-HUB**.

The main experiments investigate:

- LoRA-based parameter-efficient fine-tuning
- Native, early, and late multimodal fusion
- DenseCRF spatial refinement
- Modality and channel importance analysis

The experimental subset contains **35,850 samples** from 10 spatial regions.

## Modalities

The model uses multiple Earth observation sources:

- Aerial RGBI imagery
- nDSM elevation data
- SPOT imagery
- Sentinel-1 ascending SAR
- Sentinel-1 descending SAR
- Sentinel-2 multispectral time series

## Method

Three multimodal fusion strategies were compared:

- **Native AnySat fusion** — modality-specific projection followed by shared multimodal encoding
- **Strict early fusion** — modalities compressed before the common encoder
- **Late token-gated fusion** — modalities encoded separately and fused using learnable gates

The AnySat backbone was adapted using a two-stage fine-tuning strategy with **LoRA**.

## Results

| Method | mIoU | Mean F1 |
| --- | ---: | ---: |
| Native AnySat | **0.5964** | **0.7301** |
| Late fusion | 0.5947 | 0.7282 |
| Strict early fusion | 0.5218 | 0.6654 |
| Native AnySat + DenseCRF | **0.6028** | **0.7349** |

Late fusion achieved performance close to the native AnySat architecture, while strict early fusion caused a noticeable performance drop.

DenseCRF provided a modest improvement in spatial consistency and overall segmentation accuracy.

## Modality Analysis

Two complementary approaches were used to study multimodal contribution:

- **Ablation analysis** — measuring performance degradation after removing individual modalities or channels
- **Gate analysis** — learning modality/channel importance through trainable gates

The experiments showed that high-resolution aerial imagery provides the strongest spatial information, while Sentinel-2 contributes useful complementary spectral information.

## Tech Stack

- PyTorch
- AnySat
- LoRA
- FLAIR-HUB
- DenseCRF
- NumPy / pandas
- Matplotlib

## Repository Structure

```text
├── configs/
├── data/
├── models/
├── training/
├── evaluation/
├── analysis/
├── assets/
└── README.md
