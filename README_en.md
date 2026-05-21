<div align="center">

# Genos-m: A Foundation Model for Human-Associated Microbial Genomes

![genos_m_concept_v1.png](images/genos_m_concept_v1.png)

[![Hugging Face Model](https://img.shields.io/badge/Hugging%20Face-Genos--m-yellow?logo=huggingface)](https://huggingface.co/BGI-HangzhouAI/Genos-m)
[![Technical Report](https://img.shields.io/badge/Technical%20Report-PDF-blue)](paper/Genos-m.pdf)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](LICENSE)
[![visitors](https://hitscounter.dev/api/hit?url=https%3A%2F%2Fgithub.com%2FBGI-HangzhouAI%2FGenos-m&label=visitors&icon=github&color=%236ec044&message=&style=flat&tz=UTC)](https://hitscounter.dev/)

[English](README_en.md) ｜ [中文](README.md)

</div>

## Table of Contents

* [Model Overview](#model-overview)
* [Model and Data](#model-and-data)
  * [Training Data](#training-data)
  * [Model Architecture](#model-architecture)
* [Evaluation and Interpretability](#evaluation-and-interpretability)
  * [Evaluation Framework](#evaluation-boundary)
  * [Benchmark Results](#benchmark-results)
  * [Mechanistic Interpretability](#mechanistic-interpretability)
* [Deployment and Usage](#deployment-and-usage)
  * [Model Weights](#model-weights)
  * [Hardware and Performance](#hardware-and-performance)
  * [Quick Start](#quick-start)
* [Use Cases](#use-cases)
  * [Use-Case Overview](#use-case-overview)
  * [Case 1: Building a Microbiome Self-Supervised Learning Model with Genos-m Genome Representations](#case-1-microbiome-ssl)
  * [Case 2: A Metagenomic Individual Latent-Space Library](#case-2-metagenome-latent-space)
* [License](#license)
* [Contact](#contact)

<a id="model-overview"></a>

## Model Overview

Genome foundation models use self-supervised learning to compress large, diverse, and ultra-long DNA sequences into continuous representations. These representations allow biological sequence objects at multiple scales, including sequence fragments, whole genomes, and metagenomic samples, to be compared, searched, predicted, and reused in a unified representation space. For human-associated microbial genomes, this task requires broad prokaryotic genomic semantics and domain-specific signals from the human microbiome. Here, we introduce Genos-m, an open-source foundation model for human-associated microbial genome modeling. Unlike general-purpose DNA foundation models trained across multiple domains of life [1], Genos-m is trained primarily on human-associated microbial genomes, including prokaryotic isolate genomes, metagenome-assembled genomes (MAGs), and phage genomes. Its pre-training corpus also includes representative prokaryotic genomes from the Genome Taxonomy Database (GTDB), providing broad cross-species genomic coverage.

Genos-m inherits the core architecture of Genos [2]. It uses a sparsely activated Mixture-of-Experts (MoE) Transformer with 4.7B total parameters, approximately 330M activated parameters per forward pass, and context lengths up to 1M bp. This architecture increases effective model capacity while controlling training and inference cost. It is designed to model the cross-species variation, strain-level diversity, and long-range genomic dependencies common in human-associated microbial sequence corpora.

The Genos-m pre-training corpus was constructed through hierarchical data curation and dynamic sampling. Because microbial genome resources vary in assembly completeness, contamination, and source heterogeneity, Genos-m applies strict quality control to reduce the influence of low-quality sequences on contextual learning. The final pre-training corpus contains approximately 1.2T tokens and covers 186 phyla, 3,448 families, and 69,056 species. Within this corpus, the retained human-associated prokaryotic subset covers 45 phyla, 585 families, and 12,273 species across major human microbial habitats, including the gut, oral cavity, skin, respiratory tract, and female reproductive tract. Genos-m further uses reverse-complement sequence augmentation and multi-stage context extension to support million-base context modeling.

We evaluated Genos-m with a representation-to-task benchmark spanning local sequences, regional long-context sequences, and whole genomes. We also assessed zero-shot RNA sequence transfer, mechanistic interpretability, and task-level applications. Genos-m performs consistently across these multi-scale benchmarks and, despite its smaller activated parameter count, achieves performance comparable to or better than large general-purpose DNA models such as Evo2-40B on several tasks. In downstream applications, Genos-m improves cross-cohort colorectal cancer classification by providing genome sequence representations for species features in an abundance-based model. It also supports stable sample-level representations from low-depth metagenomic data, suggesting utility for disease-risk modeling and metagenomic representation learning.

We release Genos-m as an open-source foundation platform with model weights, training and inference code, standardized benchmarks, and deployable inference components. The release is intended to support reproducible, evaluable, and scalable model reuse, benchmarking, and method development in local or cloud environments.

<a id="model-and-data"></a>

## Model and Data

<a id="training-data"></a>

### Training Data

#### Pre-Training Data Design

The Genos-m training corpus was built with a hierarchical strategy:

1. High-quality representative genomes of GTDB prokaryotic species were included to learn core prokaryotic genomic semantics and cross-species regularities.
2. Publicly available human-associated microbial genomes were collected, covering commensal and pathogenic prokaryotes as well as phages. This focuses the model on the target domain of human-associated microbes.
3. In-house high-quality human gut microbial genomes were added to expand strain-level diversity in the corpus.

After strict quality control and dynamic stratified sampling, the final training corpus contains **1.2T tokens**.

#### Table 1. Sources and composition of the Genos-m pre-training database

| Dataset | Number of tokens (**billions**) for pre-training | Genos-m<br>Dataloader weight | Data<br>Link | References |
| --- | --- | --- | --- | --- |
| GTDB R220 (a) | 379 (67,525 prokaryotic genomes used) | 31.30% | [https://data.ace.uq.edu.au/public/gtdb/data/releases/release220/220.0/](https://data.ace.uq.edu.au/public/gtdb/data/releases/release220/220.0/) | [3] |
| Human-associated prokaryotes: UHGG (b), AWI-Gen2 (c), gcMeta (d), IMGG (e), CGR (f), HROM (g), PMDB (h) | 446 (102,986 prokaryotic genomes used) | 36.85% | [https://ftp.ebi.ac.uk/pub/databases/metagenomics/mgnify_genomes/human-gut/v2.0.2/](https://ftp.ebi.ac.uk/pub/databases/metagenomics/mgnify_genomes/human-gut/v2.0.2/)<br>[https://zenodo.org/records/14929430](https://zenodo.org/records/14929430)<br>[https://gcmeta.wdcm.org/download](https://gcmeta.wdcm.org/download)<br>[https://www.ncbi.nlm.nih.gov/bioproject/PRJNA482748](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA482748)<br>[https://www.ncbi.nlm.nih.gov/bioproject/PRJNA903559](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA903559)<br>[https://www.ncbi.nlm.nih.gov/bioproject/PRJNA763692](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA763692)<br>[https://www.decodebiome.org/HROM/listdir.php?directory=data/genome_catalog](https://www.decodebiome.org/HROM/listdir.php?directory=data/genome_catalog) | [4-9] |
| In-house human gut high-quality MAGs | 382 (89,320 prokaryotic genomes used) | 31.53% | In-house data |  |
| UHGV (i) | 3.8 (57,514 phage genomes used) | 0.32% | [https://portal.nersc.gov/UHGV/genome_catalogs/votus_hq_plus.fna.gz](https://portal.nersc.gov/UHGV/genome_catalogs/votus_hq_plus.fna.gz) | [10] |

(a) GTDB R220 comprises 584,382 bacterial and 12,477 archaeal genomes organized into 107,235 bacterial and 5,869 archaeal species clusters. A total of 67,525 species-level representative genomes were used for training.

(b) Unified Human Gastrointestinal Genome (UHGG) collection.

(c) Africa Wits-INDEPTH Partnership for Genomic Studies (AWI-Gen).

(d) Global Catalogue of Metagenomics (gcMeta).

(e) Inner Mongolian Gut Genome (IMGG).

(f) Culturable Genome Reference (CGR).

(g) Human Reference Oral Microbiome (HROM).

(h) PMDB: the curated Pathogenic Microorganism Database from BGI, including more than 40,000 pathogens across bacteria, fungi, viruses, and parasites.

(i) Unified Human Gut Virome Catalog (UHGV).

#### Quality Control and Processing

* **Quality control:** Prokaryotic genomes were retained if they were either isolate genomes or MAGs with **completeness > 90%** and **contamination < 1%** according to CheckM. Phage genomes were retained if classified by CheckV as confident genomes, including complete circular genomes or high-quality genomes with >90% completeness.
* **Data augmentation:** A **50% reverse-complement augmentation** strategy was used.
* **Length extension:** Context length was progressively extended during training across **8k / 32k / 128k / 1M** stages.
* **Sampling strategy:** Dynamic stratified sampling was performed by data source, ecological niche, and species distribution.
* **Pre-processing:** Contigs from the same genome were concatenated using the `#` character. Runs of more than three consecutive `N` characters were replaced by `@`.

#### Human-associated Microbial Diversity and Coverage

* **Taxonomic breadth:** The final pre-training corpus covers 186 phyla, 3,448 families, and 69,056 species. Within this corpus, the retained human-associated prokaryotic subset covers 45 phyla, 585 families, and 12,273 species.
* **Niche distribution:** Major human microbial habitats, including the gut, oral cavity, skin, female reproductive tract, and respiratory tract.
* **Geographic distribution:** Asia, Europe, North America, Africa, Oceania, South America, and other regions.
* **Host association:** Human commensals, opportunistic pathogens, and human-associated pathogenic microorganisms.

**Table 2. Overview of the Genos-m pre-training corpus by source, niche, geography, and dominant phyla**

| Dimension | Category | Estimated proportion |
| --- | --- | --- |
| **Dataset source** | In-house dataset | ~32% |
|  | Public datasets, including GTDB, UHGG, AWI-Gen2, gcMeta, IMGG, CGR, HROM, and PMDB | ~68% |
| **Niche distribution** | Human gut | ~88.9% |
|  | Human oral cavity | ~4.3% |
|  | Human skin | ~1.5% |
|  | Human female reproductive tract | ~0.5% |
|  | Human respiratory tract | ~4.7% |
| **Geographic distribution** | Asia, including in-house data | ~60.5% |
|  | Europe | ~14.5% |
|  | North America | ~6.2% |
|  | Africa | ~6.0% |
|  | Oceania | ~1.0% |
|  | South America | ~0.2% |
|  | Other | ~11.5% |


#### Data Quality Profile

High quality and low contamination are defining features of the Genos-m pre-training corpus. In addition to isolate genomes, the MAGs included in pre-training are concentrated in the high-quality range, with completeness >90% and contamination <1%. This contamination threshold is stricter than the commonly used <5% cutoff for high-quality genomes. The design prioritizes low-contamination training data to preserve real genomic sequence structure and reduce contaminant-derived pseudo-contexts during model training. It is intended to improve the stability, reliability, and interpretability of representation learning.

<a id="model-architecture"></a>

### Model Architecture

![Genos-m architecture.png](images/genos_m_architecture.png)

**Figure 1 | Genos-m model architecture.**  
Genos-m inherits the core MoE architecture and training paradigm of Genos [2], while systematically increasing expert capacity to better model microbial genome diversity. Specifically, the Mixture-of-Experts feed-forward network was expanded to 32 experts and uses Top-2 expert routing. This reduces the effective FFN sparsity from 25% to 6.25%.

#### Sparse Mixture-of-Experts Design

* Transformer-based Mixture-of-Experts network with Top-2 routing.
* Number of experts: 32.
* Total parameters: 4.7B.
* Activated experts: 2.
* Activated parameters: 330M.

#### Tokenizer

* Single-nucleotide tokenizer.
* `N` and special tokens are excluded from the loss.

#### Ultra-Long Context Modeling

* Rotary Position Embedding (RoPE) with a base of 50M, supporting up to **1M tokens**.
* Supports tensor, pipeline, context, data, and expert parallelism.

#### Training and Inference Stability

* Gradient clipping and expert load balancing through auxiliary loss and z-loss.
* Grouped-query attention (GQA) reduces KV-cache cost, and FlashAttention improves efficiency.
* Dynamic expert activation is used during inference, supporting million-base single-sequence inference.

**Table 3. Genos-m training architecture and model parameters**

| **Specification** | **Genos-m 4.7B** |
| --- | --- |
| **Total parameters** | **4.7B** |
| **Activated parameters** | 0.33B |
| **Architecture** | MoE |
| **Number of experts** | **32** |
| **Top-k** | 2 |
| **Layers** | 12 |
| **Attention hidden size** | 1024 |
| **Attention heads** | 16 |
| **MoE FFN hidden size** | 4096 |
| **Vocabulary** | 128 (pad) |
| **Maximum context length** | 1M |

<a id="evaluation-and-interpretability"></a>

## Evaluation and Interpretability

<a id="evaluation-boundary"></a>

### Evaluation Framework

Genos-m Benchmarks evaluate model capability at three scales: **local sequences, regional long-context sequences, and whole-genome representations**. See Table 1 in the Benchmarks documentation for the complete dataset list and task definitions. The benchmark assesses how much function- and phenotype-relevant information is carried by pre-trained representations at each scale, and how usable these representations are under standardized task settings.

<a id="benchmark-results"></a>

### Benchmark Results

Genos-m performs in the top tier across multiple task categories, ranking among the top three models in most tasks.

All supervised tasks use a unified evaluation framework. Unless otherwise stated, the backbone is frozen, final-layer sequence representations are extracted, mean pooling is used to obtain embeddings, and a lightweight MLP is trained for comparison. Classification tasks report ROC-AUC, F1, and accuracy (ACC). Regression tasks report Pearson correlation. The RNAfitness task is evaluated zero-shot, without fine-tuning. It reports the Spearman correlation between experimental fitness scores and model-derived whole-sequence next-token average log-likelihood.

#### 1. Sequence Understanding Tasks

For local sequence understanding, Genos-m ranks in the top tier across both classification and regression tasks.

**Short-sequence classification tasks include essential gene identification, promoter recognition, six-class bacterial gene classification, antibiotic-resistance gene detection, and virulence-factor detection.** Genos-m ranks among the top two models in all five tasks. It reaches state-of-the-art performance in antibiotic-resistance gene detection (AUROC = 0.9896). It also achieves AUROC values of 0.8460, 0.9548, 0.8534, and 0.9932 for promoter recognition, virulence-factor detection, essential gene identification, and six-class bacterial gene classification, respectively. These results are close to, or second only to, Evo2-40B. See Table 2 in the Benchmarks documentation for detailed local-sequence classification results.

**The regression benchmark contains eight gene-fitness subtasks across nutrient-source shifts and chemical-stress conditions.** See Table 3 in the Benchmarks documentation for the environmental grouping and biological interpretation of the gene-fitness subtasks. Genos-m reaches state-of-the-art performance in five subtasks: L-Arabinose as the sole carbon source, L-Histidine as a nutrient condition, minimal medium with glucose, pyruvate as a carbon source, and perchlorate stress. In the remaining three subtasks, it ranks second to Evo2-40B. See Table 4 in the Benchmarks documentation for detailed gene-fitness prediction results.

**For regional long-context tasks represented by biosynthetic gene clusters (BGCs),** Genos-m achieves AUROC = 0.9907 for BGC versus non-BGC binary classification, comparable to Evo2-40B (0.9911). It reaches state-of-the-art performance for BGC multilabel classification (AUROC = 0.9216), outperforming Evo2-40B (0.8951) and other baseline models. These results indicate that Genos-m regional sequence representations capture multi-gene contextual information relevant to BGC detection and type discrimination. See Table 6 in the Benchmarks documentation for detailed BGC benchmark results.

![genos_m_benchmark_by_source_menos_8k_0212.png](images/genos_m_benchmark.png)

**Figure 2 | Performance comparison between Genos-m and baseline models across benchmark tasks.**

Classification tasks report AUC, except for Fitness, which reports the mean Pearson correlation across eight gene-fitness regression tasks. The x-axis covers essential gene identification, antibiotic-resistance gene detection, six-class bacterial gene classification, gene-fitness prediction, promoter recognition, virulence-factor detection, BGC binary classification, and BGC multilabel classification.

Together, these results show that the local and long-context sequence representations learned by Genos-m support diverse gene-function tasks, including classification and condition-dependent fitness prediction.

#### 2. Mutation-Aware Tasks

This evaluation uses 13 prokaryote-related RNAfitness subsets from RNAGym, which contain RNA mutant sequences and corresponding functional measurements such as expression-level changes. See Table 5 in the Benchmarks documentation for the RNAGym prokaryote-related RNAfitness assay subset [13]. RNA sequences were first preprocessed by converting U to T. Under the zero-shot setting, **the next-token average log-likelihood of the whole sequence was used as the mutant score, and correlations with experimental measurements were computed within each assay before averaging**. This task evaluates whether the pre-trained model has internalized functional constraints relevant to RNA sequences.

**Genos-m achieves a mean Spearman correlation of 0.313, ranking second among the public comparison models.**

![rnagym_prokaryotic_rna_dms_spearman_barplot.png](images/rnagym_prokaryotic_rna_dms_spearman_barplot.png)

**Figure 3 | Zero-shot performance of Genos-m and comparison models on prokaryote-related RNAGym RNAfitness datasets.**

Model scores were computed using whole-sequence next-token average log-likelihood, and Spearman correlation was calculated against experimental fitness scores within each assay.

This result demonstrates that, despite being pre-trained exclusively on DNA sequences, Genos-m has implicitly captured information relevant to RNA functional constraints. Under zero-shot conditions without any additional training, Genos-m can effectively rank functional variants of prokaryotic RNA mutants, achieving second place among all publicly compared models. This highlights the cross-molecule transferability of DNA pre-trained representations to RNA fitness prediction tasks.

#### 3. Whole-Genome Understanding Tasks

This evaluation uses the GIDEON (Global Infectious Diseases and Epidemiology Network) strain phenotype database, which provides whole-genome sequences and basic discrete phenotype labels for bacterial strains, including Gram status, oxygen requirement, motility, spore formation, and hemolytic activity. The dataset link is `macwiatrak/bacbench-phenotypic-traits-dna` [14]. See Table 7 in the Benchmarks documentation for the labels and sample counts used here. For Genos-m, each bacterial genome was split into 1 Mbp chunks. The model first produced chunk-level embeddings, which were then averaged to obtain a genome-level embedding. During evaluation, the backbone was frozen, and an independent lightweight classifier was trained on the genome-level embedding for each phenotype. Data were split by genus into training, validation, and test sets at a 7:1:2 ratio to avoid information leakage from the same genus.

In this benchmark, Genos-m ranks in the top tier across multiple strain phenotype prediction tasks and approaches Bacformer, a model designed for genome-scale modeling.

**Table 4. Performance of Genos-m and other models on bacterial phenotype prediction tasks (AUROC, mean ± SD)**

| **Model** | **GIDEON Gram positive** | **GIDEON Anaerobe** | **GIDEON Motile** | **GIDEON Spore formation** | **GIDEON Beta hemolysis** |
| --- | --- | --- | --- | --- | --- |
| **Mistral-DNA** | 65.58 ± 6.77 | 77.21 ± 2.99 | 66.98 ± 9.54 | 52.59 ± 21.62 | **63.40 ± 15.20** |
| **DNABERT-2** | 86.88 ± 5.47 | 88.84 ± 4.07 | 76.09 ± 2.49 | 72.71 ± 16.94 | <ins>62.26 ± 16.78</ins> |
| **Nucleotide Transformer** | 88.12 ± 8.21 | 90.90 ± 1.02 | 81.63 ± 3.84 | 79.25 ± 27.78 | 60.54 ± 16.54 |
| **ProkBERT** | 97.07 ± 3.41 | 92.50 ± 1.52 | 64.59 ± 5.22 | 83.66 ± 22.84 | 53.06 ± 5.76 |
| **ESM-2** | <ins>98.48 ± 1.69</ins> | 96.14 ± 1.34 | 82.07 ± 4.29 | 89.55 ± 8.27 | 54.80 ± 10.50 |
| **ESM-C** | **99.73 ± 0.29** | 96.57 ± 2.93 | 74.70 ± 8.69 | 89.91 ± 12.13 | 54.08 ± 6.86 |
| **ProtBERT** | 97.50 ± 2.39 | 97.92 ± 1.07 | 79.50 ± 6.52 | <ins>90.68 ± 7.52</ins> | 53.71 ± 12.93 |
| **gLM2** | 96.06 ± 4.29 | 91.08 ± 1.84 | 63.98 ± 5.82 | 74.53 ± 30.76 | 50.10 ± 5.57 |
| **Bacformer** | 97.61 ± 3.16 | **98.92 ± 0.37** | **87.01 ± 9.71** | **90.85 ± 10.38** | 52.05 ± 8.24 |
| **Genos-m** | 98.41 ± 1.77 | <ins>98.14 ± 0.57</ins> | <ins>83.22 ± 3.52</ins> | 90.24 ± 14.58 | 55.50 ± 1.59 |

These results show that the sequence-level pre-trained representations of Genos-m, aggregated through simple chunk-level mean pooling, retain global information associated with basic strain phenotypes at the genome scale. Without requiring a dedicated whole-genome modeling architecture, Genos-m approaches the performance of genome-specialized models such as Bacformer, suggesting that its local sequence representations exhibit strong cross-scale composability.

<a id="mechanistic-interpretability"></a>

### Mechanistic Interpretability

To test whether Genos-m pre-trained representations contain interpretable units aligned with known genomic structures, we used sparse autoencoders (SAEs) for mechanistic interpretability analysis. SAEs decompose dense, polysemantic activations into approximately monosemantic sparse features through an overcomplete dictionary and a sparsity constraint. This approach has been used to interpret internal representations in natural language models and biological language models [11,12]. Some sparse features can correspond to specific, localizable, and verifiable sequence or structural patterns. In this study, we applied SAE-based sparse decomposition to the Genos-m output layer and aligned high-activation features with known annotated regions in genomic coordinates. This analysis tested whether interpretable genomic structural features are present in the model representation space.

Using the _E. coli_ reference genome NC_000913.3, we searched for sparse features in the SAE model. The model used hidden dimension 4096, k = 128, batch size 4096, learning rate 5 × 10^-5, and approximately 1B training tokens. Some Genos-m sparse features corresponded to known genomic structures. The model identified high-activation features associated with ORF, intergenic, tRNA, and rRNA regions, and distinguished positive-strand from negative-strand ORFs. **These findings indicate that Genos-m hidden activations contain sparse features that can be mapped to known genomic structures, providing preliminary mechanistic interpretability evidence for its sequence representations.**

#### Method Summary

##### SAE model training

The training data consisted of approximately 305,000 randomly sampled chunks of length 32k, all drawn from the model pre-training data. Bacteroidota was the dominant phylum in this sample set, accounting for 32.3%. These chunks were processed by Genos-m to extract last-layer activations, which were then downsampled by 1/10 to reduce redundancy. The resulting activations were globally shuffled so that tokens within any 32k window did not originate from the same raw fragment. The final SAE training set contained approximately 1B tokens.

Given an activation vector `x`, the SAE encodes it through a single-hidden-layer wide MLP into a sparse feature vector `f`, and reconstructs `x_hat`:

$$
f = \sigma(W_{\mathrm{enc}} x + b_{\mathrm{enc}})
$$

$$
\hat{x} = W_{\mathrm{dec}} f + b_{\mathrm{dec}}
$$

The feature dimension is scaled by expansion factor `E`:

$$
d_{\mathrm{SAE}} = d_{\mathrm{model}} \cdot E
$$

In this work, `d_SAE = 4096`. The nonlinear activation `sigma` uses Batch-TopK, retaining only the largest `kB` elements across a batch of size `B`, with `k = 128`. The loss is the sum of reconstruction loss and auxiliary loss:

$$
L = \lVert x - \hat{x} \rVert_2^2 + \alpha L_{\mathrm{aux}}
$$

Here, `L_aux` penalizes dead features. Training used batch size 4096, learning rate 5 × 10<sup>-5</sup>, and one epoch.

##### Feature search and evaluation

To identify SAE sparse features associated with known genomic structures, we selected clearly annotated ORF, intergenic, tRNA, and rRNA regions. For each feature, we calculated mean activation within target regions and compared it with background regions using a one-vs-rest strategy. Candidate feature-to-annotation alignment was quantified using the Domain F1-score. Target regions were treated as positives. Precision was defined as the proportion of high-activation sites that fell within the target region, and recall was defined as the proportion of the target region covered by high-activation sites. Their harmonic mean gave the Domain F1-score. No fixed activation threshold was imposed. Instead, thresholds were swept across the activation range of each feature, and the threshold that maximized the Domain F1-score was selected as the optimal threshold.

##### Main results

* High-activation features associated with ORFs were identified, including features that distinguish positive- and negative-strand orientation.
* Features associated with intergenic, tRNA, and rRNA regions were identified.

These results indicate that the model contains separable internal features associated with known biological structures. See the [paper PDF](paper/Genos-m.pdf) for full methods and results.

![image.png](images/SAE-image-1.png)

**Figure 4 | SAE analysis resolves semantic features of prokaryotic genomes.**

**Open-source plan:** SAE model weights, SAE training code, and feature-search code are planned for release with the project.

<a id="deployment-and-usage"></a>

## Deployment and Usage

<a id="model-weights"></a>

#### Model Weights

| Model | Total parameters | Hugging Face | Megatron ckpt |
| --- | --- | --- | --- |
| Genos-m | 4.7B | [Genos-m-4.7B](https://huggingface.co/BGI-HangzhouAI/Genos-m) | [Genos-m-Megatron-4.7B](https://huggingface.co/BGI-HangzhouAI/Genos-m-Megatron) |

<a id="hardware-and-performance"></a>

### Hardware and Performance

The following results were benchmarked on a single **NVIDIA A800 80GB HBM2e** GPU. Genos-m-4.7B uses approximately **9 GB** of VRAM for model weights and supports **1M bp** context inference on one GPU. On a 24 GB consumer GPU, Genos-m-4.7B can run approximately **200k bp** context inference, covering most microbial gene, gene-cluster, and local genomic-neighborhood analysis scenarios.

<img style="display: block; margin: 0 auto;" src="images/Genos-m-inference-performance.png" alt="Genos-m inference performance" width="50%">

<a id="quick-start"></a>

### Quick Start

The following examples show two common inference modes: extracting sequence embeddings and generating sequence continuations.

```bash
pip install torch transformers accelerate
```

#### Extracting Sequence Embeddings

```python
import torch
from transformers import AutoModel, AutoTokenizer

model_id = "BGI-HangzhouAI/Genos-m"

tokenizer = AutoTokenizer.from_pretrained(model_id, trust_remote_code=True)
model = AutoModel.from_pretrained(
    model_id,
    torch_dtype=torch.bfloat16,
    device_map="auto",
    trust_remote_code=True,
).eval()

sequence = "ATGCGTACGTAGCTAGCTAGCTAGCTAGCTAA"
inputs = tokenizer(
    sequence,
    return_tensors="pt",
    truncation=True,
    max_length=8192,
).to(model.device)

with torch.no_grad():
    outputs = model(**inputs)

hidden_states = outputs.last_hidden_state
attention_mask = inputs["attention_mask"].unsqueeze(-1)
embedding = (hidden_states * attention_mask).sum(dim=1) / attention_mask.sum(dim=1)

print(embedding.shape)
```

#### Generation

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "BGI-HangzhouAI/Genos-m"

tokenizer = AutoTokenizer.from_pretrained(model_id, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    torch_dtype=torch.bfloat16,
    device_map="auto",
    trust_remote_code=True,
).eval()

prompt = "ATGCGTACGTAGCTAGC"
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
pad_token_id = tokenizer.pad_token_id
if pad_token_id is None:
    pad_token_id = tokenizer.eos_token_id

with torch.no_grad():
    output_ids = model.generate(
        **inputs,
        max_new_tokens=256,
        do_sample=True,
        temperature=0.8,
        top_p=0.95,
        pad_token_id=pad_token_id,
    )

generated_ids = output_ids[0, inputs["input_ids"].shape[-1]:]
generated_sequence = tokenizer.decode(generated_ids, skip_special_tokens=True)

print(prompt + generated_sequence)
```

<a id="use-cases"></a>

## Use Cases

<a id="use-case-overview"></a>

### Use-Case Overview

This section describes two representative applications of Genos-m: genome-informed microbiome community modeling and low-depth metagenomic sample representation.

<a id="case-1-microbiome-ssl"></a>

### Case 1: Building a Microbiome Self-Supervised Learning Model with Genos-m Genome Representations

#### Case Summary

This case demonstrates how Genos-m can be used for self-supervised modeling of human microbiome communities. Genos-m first generates genome-level embeddings for representative genomes of each species. These continuous vectors are projected and aligned with species tokens, then used as genome-informed species representations in a microbiome self-supervised learning model. The model is a Transformer-based self-supervised human microbial community model. It has 30 layers, hidden size 1792, and 14 attention heads. It takes species relative-abundance profiles as input and is trained on approximately 400k unlabeled human microbiome samples to learn species composition, co-occurrence relationships, and abundance structure across the gut and other body sites.

![Microbiome self-supervised learning model architecture](images/microbiome_ssl_model_architecture.png)

**Figure 5 | The microbiome self-supervised learning model architecture.**


#### Model Architecture

Existing self-supervised models for microbiome abundance data [27,28] typically treat species as discrete tokens and mainly model species-level co-occurrence and abundance structure. However, species tokens do not directly contain genomic sequence information, making it difficult to introduce functional genomic signals and inter-species sequence differences explicitly. Genos-m provides a complementary genome-level representation in this framework. By encoding representative genome sequences as continuous vectors, it allows the community model to incorporate genome-informed species representations in addition to species abundance structure.

In the microbiome self-supervised learning model, each species position receives two inputs: a learnable species-token embedding and a genome-level embedding generated in advance by Genos-m. The two inputs are linearly projected and summed.

The backbone is adapted from the [slowrun](https://github.com/qlabs-eng/slowrun) implementation of nanoGPT. It is designed to improve training effectiveness under a limited data budget of 100M tokens. Compared with native nanoGPT, it incorporates Interleaved Head Attention (IHA), RMSNorm, RoPE, U-Net-style skip connections, and several training-strategy optimizations to improve stability. The model uses abundance-sorted causal language modeling: species in each sample are sorted by decreasing relative abundance, and the model predicts subsequent species from preceding species. This objective enables the model to learn microbiome community composition and abundance-dependent relationships.

#### Evaluation Design

The downstream evaluation follows the data and methodology of Piccinno et al. [26]. It uses their publicly curated **14 independent CRC gut metagenomic cohorts**[15-26], covering populations from Asia, Europe, and North America. The task uses the species relative-abundance Random Forest classifier reported by Piccinno et al. as the baseline and compares the microbiome self-supervised learning model under the same cohort settings for CRC discrimination. During classification, the self-supervised backbone is frozen and only a lightweight MLP classification head is trained, providing a direct test of downstream representation usability. The evaluation includes:

1. **Within-cohort cross-validation:** Cross-validation within each cohort to evaluate within-cohort classification performance.
2. **Cross-cohort external validation:** Training on one cohort and testing on other cohorts to evaluate generalization across populations and studies.

#### Key Results

Across 14 global CRC cohorts, the microbiome self-supervised learning model with Genos-m representations achieved a mean AUROC of 0.89 in within-cohort 10-fold cross-validation, outperforming the traditional species relative-abundance Random Forest baseline (AUROC = 0.86, **ΔAUROC = +0.03**). In cross-cohort external validation, the model achieved a mean AUROC of 0.77, again outperforming the corresponding Random Forest baseline (AUROC = 0.72, **ΔAUROC = +0.05**). These results show that the microbiome self-supervised learning model improves over traditional abundance-based models in both within-cohort and cross-cohort settings, with a larger gain under cross-cohort external validation. The findings suggest that adding genome-informed species representations beyond abundance structure can improve within-cohort discrimination and cross-cohort generalization for CRC abundance-based classification.

| **Method** | **Mean within-cohort AUROC** | **Mean cross-cohort AUROC** |
| --- | --- | --- |
| Piccinno et al. (RF baseline) [26] | <ins>0.86</ins> | <ins>0.72</ins> |
| **Genos-m + microbiome self-supervised learning model** | **0.89** | **0.77** |

<img style="display: block; margin: 0 auto;" src="images/microbiome_ssl_model_auc_delta.png" alt="Microbiome SSL model AUC delta heatmap" width="50%" >

**Figure 6 | Heat map of AUROC differences between the microbiome self-supervised learning model and Random Forest in cross-study tasks.** The y-axis (Training) indicates the training dataset, and the x-axis (Testing) indicates the test dataset. Diagonal cells with borders represent within-dataset cross-validation, while off-diagonal cells represent cross-study generalization. Each cell shows the AUROC difference between the two methods, defined as `AUROC_model - AUROC_RF`. Positive values in warm colors indicate performance gains from the microbiome self-supervised learning model, while negative values in cool colors indicate performance below the baseline. The matrix diagonal shows within-cohort cross-validation performance, and off-diagonal entries show cross-cohort transfer performance.

#### Conclusion

This case shows that Genos-m can serve as a **genome-representation enhancement module** for microbiome self-supervised learning models through projection, alignment, and sequence modeling. Across 14 CRC cohorts, the microbiome self-supervised learning model with Genos-m representations achieved higher AUROC than traditional species relative-abundance models in both within-cohort cross-validation and cross-cohort external validation. This indicates that genome sequence representations can provide complementary information beyond abundance features for disease-associated microbiome signal modeling. Similar genome-informed species representations can be extended to other microbiome modeling frameworks based on species-level inputs.

<a id="case-2-metagenome-latent-space"></a>

### Case 2: A Metagenomic Individual Latent-Space Library

An individual latent-space library is a standardized collection of individual-level representations produced by applying Genos-m directly to metagenomic reads within each sample. Each sample is mapped to a continuous, stable, and comparable vector space.

#### Objective

The central objective of this case is to evaluate **the ability of Genos-m to generate sample-level representations from low-depth metagenomic reads**. We test whether these representations preserve community-composition information and remain robust under low-input conditions. The goal is to provide a unified sample-level embedding basis for metagenomic sample retrieval, clustering, similarity comparison, and downstream quality-control analysis.

#### Methods

We selected human gut metagenomic sequencing datasets covering multiple continents and multiple Asian sources. Raw paired-end reads were processed with a unified procedure: read 2 was reverse-complemented and concatenated with read 1 to form read-pair-level sequence inputs. Each sample was then downsampled to 10K, 100K, 1M, and 10M reads, with five independent replicates at each depth. This design simulated low-depth scenarios and evaluated representation robustness. We also constructed simulated datasets by mixing two samples at 10% increments to test how Genos-m sample representations respond to perturbations in community composition.

During inference, Genos-m directly generated embeddings for metagenomic read-pair sequences. Read-level embeddings were aggregated into sample-level embeddings using mean pooling. **To evaluate the biological information retained by low-depth sample-level representations**, we designed two tasks. First, under the 10K-read setting, we evaluated whether metagenomic sample embeddings could recover host geographic origin (**Table 6**). Second, we generated genus-level abundance profiles from the full reads using Kraken2 and used PAM-JSD (partitioning around medoids with Jensen–Shannon divergence) to define reference enterotypes. We then evaluated whether low-depth Genos-m embeddings could recover gut community structure, as represented by enterotypes. As a comparison, we performed enterotype clustering using genus-level abundance profiles under the same 10K-read condition. This comparison tested biological information retention and community-structure resolution between Genos-m representations and conventional abundance data in a low-depth sampling scenario.

**Table 6. Geographic distribution of samples**

| Region | Number of samples | Data source |
| --- | --- | --- |
| Northeast China | 2033 | In-house samples |
| East China | 500 | [29] |
| South China | 500 | [30] |
| Europe | 500 | [15-16,20-21,26] |
| North America | 104 | [17] |
| Africa | 500 | [31-32] |

#### Key Results

**Stage 1 | Stability and perturbation response of low-depth read-based sample representations**

Across downsampling depths from 10K to 10M reads, Genos-m sample-level embeddings show high depth-independent stability. Representations from the same sample at different depths cluster tightly in the latent space (Figure 7a), **indicating that the model preserves stable sample-specific signals even under low-read input conditions** and that the global representation structure does not shift substantially with sequencing depth. In the two-sample 10% incremental mixture test, sample embeddings form continuous trajectories in UMAP space as community read-composition proportions change (Figure 7b). These results indicate that Genos-m sample-level representations combine within-sample stability with sensitivity to between-sample community-composition changes. This supports their use for low-depth metagenomic sample similarity comparison, perturbation detection, and community-structure analysis.

![Low-depth stability and mixture response](images/case2-image-1.png)

**Figure 7 | Stability and mixture-perturbation evaluation of low-depth Genos-m sample-level representations.**

**Stage 2 | Low-depth read-based sample representations retain host geographic origin information**

Low-depth sample embeddings generated from 10K reads accurately distinguish human gut metagenomic samples from different geographic origins. Samples from different continents and regional sources form relatively separated clusters in UMAP space (**Figure 8**), suggesting that host-geography-related gut community-structure features are retained in low-depth embeddings. A supervised linear classifier trained on these sample embeddings reaches AUC = 0.998 for geographic-origin prediction. This result shows that Genos-m sample-level representations retain host-geographic features that are usable by downstream models under low-input conditions, **providing a feasible representation basis for low-depth metagenomic source discrimination, cohort-structure inspection, and batch or geographic stratification analysis.**

<img style="display: block; margin: 0 auto;" src="images/case2-image-2.png" alt="Geographic source recovery from low-depth embeddings" width="50%" >

**Figure 8 | Low-depth Genos-m sample-level representations distinguish human gut samples from different geographic origins.**

**Stage 3 | Low-depth Genos-m sample-level representations retain enterotype-related community structure**

Unsupervised clustering of Genos-m sample-level representations generated from 10K reads forms two clusters (**Figure 9a**) that are highly consistent with the reference enterotypes ET_P and ET_B defined from full-read genus-level abundance profiles (**Figure 9b; consistency: 86%**). As a control, clustering based on Kraken2 genus-level abundance matrices under the same 10K-read condition shows clear crossover and mismatch against the full-read reference enterotypes (**Figure 9c**). These results indicate that, in low-input metagenomic data scenarios, Genos-m sample-level representations better preserve enterotype-related community structure and reduce structural bias caused by increased sparsity in conventional abundance matrices at low sequencing depth.

![Low-depth embedding clusters and enterotype consistency](images/case2-image-3.png)

**Figure 9 | Low-depth Genos-m sample-level representations stably recover enterotype structure defined from full-depth abundance profiles.**

**a** Unsupervised clustering of Genos-m sample-level representations under the 10K-read condition.  

**b** Correspondence between low-depth Genos-m representation clusters and the reference binary enterotypes ET_P and ET_B derived from full-read genus-level abundance profiles. ET_P is dominated by _Prevotella_ abundance, and ET_B is dominated by _Bacteroides_ abundance.  

**c** Correspondence between Kraken2 genus-level abundance clustering under the same 10K-read condition and the full-read reference enterotypes.

#### Conclusion

This case shows that **Genos-m can generate sample-level representations directly from metagenomic reads under low sequencing input and retain community-structure information**, including sample identity, geographic origin, and enterotype structure. Downsampling experiments show that Genos-m representations of the same sample remain highly consistent in latent space across read depths, indicating stable individual-specific features under low-input conditions. Mixture-perturbation experiments further show that sample representations change continuously with community-composition proportions, suggesting detectable responsiveness to community shifts.

Geographic-origin discrimination and enterotype evaluation show that, with only 10K reads, Genos-m representations can capture gut microbiome differences associated with host geography and recover enterotype structure defined from full-depth metagenomic data. Compared with conventional genus-level abundance clustering under the same low-depth condition, Genos-m representations show stronger enterotype recovery (**86% vs 54% consistency**). These results support Genos-m as a low-depth and low-cost metagenomic sample representation tool for sample similarity comparison, clustering, sample-structure inspection, and quality-control assistance.

#### Scope and Limitations

Low-depth Genos-m sample-level representations are suitable for rapid assessment of multi-source metagenomic sample collections. With a small number of reads, they can define a sample embedding space for similarity comparison, major group-structure identification, and assisted flagging of outlier samples, batch effects, sample mixing, or potential contamination risks. These representations can also serve as sample-level input features for downstream statistical analyses or machine-learning models. This case evaluates sample-level representation capability, not taxonomic abundance estimation. Conventional taxonomic profiling remains the standard procedure for obtaining interpretable species composition and relative-abundance profiles. Genos-m provides a complementary sample-level latent representation that captures sequence-level sample information beyond conventional abundance profiles.

<a id="license"></a>

## License

* Model and code are released under the [Apache License 2.0](LICENSE).

<a id="contact"></a>

## Contact

* Questions and suggestions: please submit an Issue.

## References

1. Brixi G, Durrant M G, Ku J, et al. Genome modeling and design across all domains of life with Evo 2[J]. Nature, 2026, 652(8112): 1349-1361.
2. Lin A, Xie B, Ye C, et al. Genos: a human-centric genomic foundation model[J]. GigaScience, 2025, 14: giaf132.
3. Parks D H, Chuvochina M, Rinke C, et al. GTDB: an ongoing census of bacterial and archaeal diversity through a phylogenetically consistent, rank normalized and complete genome-based taxonomy[J]. Nucleic acids research, 2022, 50(D1): D785-D794.
4. Almeida A, Nayfach S, Boland M, et al. A unified catalog of 204,938 reference genomes from the human gut microbiome[J]. Nature biotechnology, 2021, 39(1): 105-114.
5. Maghini D G, Oduaran O H, Olubayo L A I, et al. Expanding the human gut microbiome atlas of Africa[J]. Nature, 2025, 638(8051): 718-728.
6. Sun Y, Chen Q, Fan G, et al. gcMeta 2025: a global repository of metagenome-assembled genomes enabling cross-ecosystem microbial discovery and function research[J]. Nucleic Acids Research, 2026, 54(D1): D724-D733.
7. Jin H, Quan K, He Q, et al. A high-quality genome compendium of the human gut microbiome of Inner Mongolians[J]. Nature Microbiology, 2023, 8(1): 150-161.
8. Zou Y, Xue W, Luo G, et al. 1,520 reference genomes from cultivated human gut bacteria enable functional microbiome analyses[J]. Nature biotechnology, 2019, 37(2): 179-185.
9. Cha J H, Kim N, Ma J, et al. A high-quality genomic catalog of the human oral microbiome broadens its phylogeny and clinical insights[J]. Cell Host & Microbe, 2025, 33(11): 1977-1994. e8.
10. Camargo A P, Baltoumas F A, Ndela E O, et al. A genomic atlas of the human gut virome elucidates genetic factors shaping host interactions[J]. bioRxiv, 2025: 2025.11. 01.686033.
11. MAIWALD A, JEDRYSZEK P, DRAYE F, et al. Decode-gLM: Tools to interpret, audit, and steer genomic language models[J/OL]. bioRxiv, 2025[2026-04-30]. [https://doi.org/10.1101/2025.10.31.685860](https://doi.org/10.1101/2025.10.31.685860).
12. ORLOV A V, MAKUS Y V, ASHNIEV G A, et al. What do biological foundation models compute? Sparse autoencoders from feature recovery to mechanistic interpretability[J/OL]. bioRxiv, 2026[2026-04-30]. [https://doi.org/10.1101/2026.03.04.709491](https://doi.org/10.1101/2026.03.04.709491).
13. Arora R, Angelo M, Choe C A, et al. RNAGym: Large-scale Benchmarks for RNA Fitness and Structure Prediction[J/OL]. _bioRxiv_, 2025. DOI: 10.1101/2025.06.16.660049.
14. Wiatrak M, Weimann A, Viñas Torné R, et al. BacBench: Evaluating Genomic Language Models for Bacteria[EB/OL]. OpenReview, ICLR 2026 Conference Withdrawn Submission, 2025[2026-03-18].
15. Zeller G, et al. Potential of fecal microbiota for early-stage detection of colorectal cancer[J]. Molecular Systems Biology, 2014, 10: 766.
16. Feng Q, et al. Gut microbiome development along the colorectal adenoma-carcinoma sequence[J]. Nat Commun, 2015, 6: 6528.
17. Vogtmann E, et al. Colorectal Cancer and the Human Gut Microbiome: Reproducibility with Whole-Genome Shotgun Sequencing[J]. PLoS ONE, 2016, 11(5): e0155362.
18. Yu J, et al. Metagenomic analysis of faecal microbiome as a tool towards targeted non-invasive biomarkers for colorectal cancer[J]. Gut, 2017, 66: 70-78.
19. Gupta A, et al. Association of Flavonifractor plautii, a Flavonoid-Degrading Bacterium, with the Gut Microbiome of Colorectal Cancer Patients in India[J]. mSystems, 2019, 4: e00438-19.
20. Thomas A M, et al. Metagenomic analysis of colorectal cancer datasets identifies cross-cohort microbial diagnostic signatures and a link with choline degradation[J]. Nat Med, 2019, 25: 667-678.
21. Wirbel J, et al. Meta-analysis of fecal metagenomes reveals global microbial signatures that are specific for colorectal cancer[J]. Nat Med, 2019, 25: 679-689.
22. Yachida S, et al. Metagenomic and metabolomic analyses reveal distinct stage-specific phenotypes of the gut microbiota in colorectal cancer[J]. Nat Med, 2019, 25: 968-976.
23. Yang J, et al. Establishing high-accuracy biomarkers for colorectal cancer by comparing fecal microbiomes in patients with healthy families[J]. Gut Microbes, 2020, 11: 918-929.
24. Yang Y, et al. Dysbiosis of human gut microbiome in young-onset colorectal cancer[J]. Nat Commun, 2021, 12: 6757.
25. Liu N-N, et al. Multi-kingdom microbiota analyses identify bacterial-fungal interactions and biomarkers of colorectal cancer across cohorts[J]. Nat Microbiol, 2022, 7: 238-250.
26. Piccinno G, et al. Pooled analysis of 3,741 stool metagenomes from 18 cohorts for cross-stage and strain-level reproducible microbial biomarkers of colorectal cancer[J]. Nat Med, 2025, 31: 2416-2429.
27. Zhang H, Zhang Y, Kang Z, et al. MGM as a large-scale pretrained foundation model for microbiome analyses in diverse contexts[J]. Advanced Science, 2026, 13(24): e13333.
28. Pope Q, Varma R, Tataru C, et al. Learning a deep language model for microbiomes: The power of large scale unlabeled microbiome data[J]. PLOS Computational Biology, 2024, 20(1): e1011353.
29. Wu C, Yang F, Zhong H, et al. Obesity-enriched gut microbe degrades myo-inositol and promotes lipid absorption[J]. Cell Host & Microbe, 2024, 32(8): 1301-1314.e9.
30. Jie Z, Liang S, Ding Q, et al. A transomic cohort as a reference point for promoting a healthy human gut microbiome[J]. Medicine in Microecology, 2021, 8: 100039.
31. Lokmer A, Cian A, Froment A, et al. Use of shotgun metagenomics for the identification of protozoa in the gut microbiota of healthy individuals from worldwide populations with various industrialization levels[J]. PloS one, 2019, 14(2): e0211139.
32. Tett A, Huang K D, Asnicar F, et al. The Prevotella copri complex comprises four distinct clades underrepresented in westernized populations[J]. Cell host & microbe, 2019, 26(5): 666-679. e7.
