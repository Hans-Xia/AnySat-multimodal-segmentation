# Multimodal Remote Sensing Segmentation with AnySat

[English](#english) | [中文](#中文)

## English
Multimodal land-cover semantic segmentation on FLAIR-HUB using the AnySat Earth observation foundation model.

This project was developed as my MSc thesis at the Budapest University of Technology and Economics (BME), focusing on parameter-efficient fine-tuning, multimodal fusion, and modality importance analysis.

## Overview

The project adapts the pretrained **AnySat** foundation model to a 15-class land-cover segmentation task using multimodal remote sensing data from **FLAIR-HUB**.

The main experiments investigate:

- LoRA-based parameter-efficient fine-tuning
- Feature-level, early, and late multimodal fusion
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

- **Feature-level fusion with attention** — modality-specific projection followed by shared multimodal encoding
- **Strict early fusion** — modalities compressed before the common encoder
- **Late token-gated fusion** — modalities encoded separately and fused using learnable gates

The AnySat backbone was adapted using a two-stage fine-tuning strategy with **LoRA**.

## Results

| Method | mIoU | Mean F1 |
| --- | ---: | ---: |
| Feature-level fusion | **0.5964** | **0.7301** |
| Late fusion | 0.5947 | 0.7282 |
| Strict early fusion | 0.5218 | 0.6654 |
| Native AnySat + DenseCRF | **0.6028** | **0.7349** |

Late fusion achieved performance close to the feature-level fusion, and both are superior to AnySat's 0.556 mIoU on the FLAIR original dataset. However, the later fusion calculation in the experiment is more costly, while the strict early fusion shows a significant performance decline.

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

## Project Context

MSc thesis project at Budapest University of Technology and Economics.

A journal manuscript based on this work is currently in preparation.

## 中文
# 基于 AnySat 的多模态遥感语义分割

基于 FLAIR-HUB 数据集与 AnySat 地球观测基础模型开展多模态地表覆盖语义分割研究。

本项目为布达佩斯技术与经济大学（BME）计算机科学硕士毕业论文工作，主要研究参数高效微调、多模态融合以及模态重要性分析。

## 项目概述

项目将预训练的 **AnySat** 基础模型适配到一个 15 类地表覆盖语义分割任务，并使用 **FLAIR-HUB** 中的多源遥感数据开展实验。

主要研究内容包括：

- 基于 LoRA 的参数高效微调
- 特征级注意力融合、早期融合 和 后期融合 三种多模态融合策略
- DenseCRF 空间后处理
- 模态与通道重要性分析

实验子集包含来自 10 个空间区域的 **35,850 个样本**。

## 输入模态

模型融合了多种地球观测数据：

- 航空 RGBI 影像
- nDSM 高程数据
- SPOT 光学影像
- Sentinel-1 升轨 SAR
- Sentinel-1 降轨 SAR
- Sentinel-2 多光谱时间序列

## 方法

项目比较了三种多模态融合策略：

- **特征级注意力融合**：各模态先独立投影，再进入共享多模态编码过程
- **严格早期融合**：不同模态在进入公共编码器前压缩为共享表示
- **后期 Token 门控融合**：各模态先独立编码，再通过可学习门控机制进行融合

AnySat 主干网络采用两阶段训练方式，并使用 **LoRA** 实现参数高效微调。

## 实验结果

| 方法 | mIoU | Mean F1 |
| --- | ---: | ---: |
| 特征级注意力融合 | **0.5964** | **0.7301** |
| 后期融合 | 0.5947 | 0.7282 |
| 严格早期融合 | 0.5218 | 0.6654 |
| AnySat + DenseCRF | **0.6028** | **0.7349** |

后期融合的表现与 特征级融合 接近，二者皆优于 AnySat 在FLAIR原数据集上的 0.556 mIoU。但实验中后期融合计算花费更高，而严格早期融合则出现了明显的性能下降。

DenseCRF 能够小幅提升空间一致性以及整体分割性能。

## 模态分析

项目采用两种互补方式分析不同输入模态的贡献：

- **消融分析**：移除单个模态或通道，观察性能下降幅度
- **门控分析**：通过可学习门控权重估计模态和通道的重要性

实验结果表明，高分辨率航空影像提供了最主要的空间信息，而 Sentinel-2 则提供了有价值的补充光谱信息。

## 技术栈

- PyTorch
- AnySat
- LoRA
- FLAIR-HUB
- DenseCRF
- NumPy
- pandas
- Matplotlib

## 项目背景

布达佩斯技术与经济大学（BME）计算机科学硕士毕业论文项目。

目前正基于该工作准备期刊论文，源码暂不开放。
