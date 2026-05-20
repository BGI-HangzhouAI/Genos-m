# Genos-m Model Performance Benchmarks

Genos-m Benchmarks is an evaluation suite designed for Genos-m as a prokaryotic microbial genome foundation model. The suite covers tasks at multiple sequence scales, from short regulatory elements, gene-level sequences, and long regional contexts to whole-genome representations. It evaluates downstream utility in local sequence understanding, regional and system-level long-context modelling, and whole-genome embedding-based inference. It also includes an RNAfitness zero-shot task to assess whether a DNA-pretrained model provides ranking signals in an external RNA variant-effect benchmark.

The local sequence, gene-level, RNAfitness, and whole-genome phenotype tasks are mainly adopted or adapted from public evaluation resources, including BacBench, GenerAnno, RNAGym, MIBiG, and related public databases.

## Evaluation Overview

Genos-m Benchmarks is organized around three core evaluation axes. Together, they provide systematic coverage from short-sequence representations to whole-genome embeddings.

**1. Local Sequence and Gene-Level Understanding**

These tasks focus on short fragments, genes, and functional elements. They evaluate whether the model captures local biological features, regulatory signals, functional classes, and sequence patterns associated with environmental adaptation.

- Essential gene identification
- Promoter recognition
- Six-class bacterial gene classification
- Antibiotic-resistance gene detection
- Virulence-factor detection
- Gene-fitness prediction
- RNAfitness zero-shot evaluation

**2. Regional and System-Level Long-Context Understanding**

These tasks focus on multi-gene modules and long genomic regions. They evaluate whether the model can identify structured functional regions, integrate neighboring-gene information, and capture regional functional organization in longer genomic contexts.

- BGC binary classification
- BGC multilabel classification

**3. Whole-Genome Understanding**

This task uses whole-genome embeddings to evaluate whether the model can recover genome-scale signals that support lineage-aware phenotype inference.

- Strain phenotype prediction

## Main Features

- **Biologically diverse task coverage.** The benchmark includes regulatory-element recognition, gene-region and functional classification, BGC region detection and type annotation, condition-specific gene-fitness regression, and strain phenotype prediction. These tasks test whether pretrained representations retain usable signals related to function, regional organization, and phenotype.
- **Multi-scale sequence evaluation.** The suite spans short regulatory elements of tens of base pairs, gene-level sequences, regional long-context inputs, and whole-genome sequences.
- **Standardized downstream evaluation.** Model representations are evaluated with lightweight MLP downstream classifiers or regressors under unified data splits and metrics. For classification tasks, MLP results are summarized by AUC, ACC, and F1 in Table S1.
- **Data partitioning and leakage control.** Depending on the original benchmark setting, tasks use genus-level separation, stratified random splits, or k-fold cross-validation. For tasks with lineage leakage risk, genus-level or phylogenetically separated splits are preferred.

## Datasets and Task Definitions

Table 1 summarizes the tasks, data sources, split strategies, and biological interpretation covered by this benchmark. It defines the scope of each reported result and helps avoid interpreting single-gene, regional, and whole-genome tasks as evidence for the same model capability.

Unless otherwise specified, sequence length refers to the length of the input nucleotide fragment. For each model, inputs are truncated according to its maximum context length when needed. Task-specific details are provided in the evaluation protocols below.

**Table 1. Dataset inventory and task definitions in Genos-m Benchmarks**

| Dataset | Source | Task | Task type | Size | Split | Sequence length | Biological meaning |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Local sequence understanding: classification tasks** |  |  |  |  |  |  |  |
| `macwiatrak/bacbench-essential-genes-dna` | DEG / GenBank | Essential gene identification | Binary classification | 169,408 | Genus-level split | avg 1 kb | Tests whether the model recognizes sequence features associated with bacterial gene essentiality. The genus-level split is used to reduce lineage leakage. |
| `promoter_ipro_mp` | Prokaryotic Promoter Database | Promoter recognition | Binary classification | 321,858 | 8:2 | 81 bp | Tests whether the model captures short regulatory elements and local sequence grammar in extremely short inputs. |
| `GenerTeam/prokaryotic-gener-tasks::gene_classification_bacteria` | RefSeq | Six-class bacterial gene classification | Six-class classification | 30k | 9:1 | avg 837 bp | Tests discrimination among CDS, pseudogene, tRNA, rRNA, ncRNA, and intergenic regions. |
| `GenerTeam/prokaryotic-gener-tasks::drug_resistence_prediction` | CARD / RefSeq | Antibiotic-resistance gene detection | Binary classification | 16.1k | 8:2 | 120 bp-4.5 kb | Tests whether gene-level sequence representations capture resistance-associated functional signals. |
| `virulence_gene_vfdb_coregenes` | VFDB / core-gene negatives | Virulence-factor detection | Binary classification | 9,559 | 8:2 | <8 kb | Tests whether the model identifies virulence-related functional genes. Positive examples are representative experimentally supported VFDB genes. Negative examples are DEG essential genes after removing annotations suggestive of toxins, virulence factors, secretion systems, effectors, capsules, polysaccharides, and outer-structure genes. |
| **Local sequence understanding: regression tasks** |  |  |  |  |  |  |  |
| `GenerTeam/prokaryotic-gener-tasks::fitness_Min-media-glucose` | Fitness Browser | Gene-fitness prediction | Regression | 35.6k | 9:1 | avg 1002 bp | Predicts gene-fitness scores in minimal medium with glucose. This tests representation of basal carbon-source metabolism. |
| `GenerTeam/prokaryotic-gener-tasks::fitness_L-Arabinose_C` | Fitness Browser | Gene-fitness prediction | Regression | 60k | 9:1 | avg 1030 bp | Predicts fitness when L-arabinose is the sole carbon source. This tests representation of specific carbon-source metabolism. |
| `GenerTeam/prokaryotic-gener-tasks::fitness_Pyruvate_C` | Fitness Browser | Gene-fitness prediction | Regression | 60k | 9:1 | avg 1013 bp | Predicts fitness when pyruvate is used as the carbon source. This tests modelling of central energy metabolism. |
| `GenerTeam/prokaryotic-gener-tasks::fitness_D-Alanine_N` | Fitness Browser | Gene-fitness prediction | Regression | 28.1k | 9:1 | avg 1015 bp | Predicts fitness when D-alanine is used as a nitrogen source. This tests adaptation to specific nitrogen metabolism. |
| `GenerTeam/prokaryotic-gener-tasks::fitness_Ammonium-chloride_N` | Fitness Browser | Gene-fitness prediction | Regression | 37.6k | 9:1 | avg 1004 bp | Predicts fitness when ammonium chloride is used as a nitrogen source. This tests signals related to nitrogen utilization and regulation. |
| `GenerTeam/prokaryotic-gener-tasks::fitness_L-Histidine_nutrient` | Fitness Browser | Gene-fitness prediction | Regression | 35.6k | 9:1 | avg 1002 bp | Predicts fitness when L-histidine is used as a nutrient. This tests representation of complex nutrient regulation. |
| `GenerTeam/prokaryotic-gener-tasks::fitness_Cisplatin_stress` | Fitness Browser | Gene-fitness prediction | Regression | 20.9k | 9:1 | avg 1028 bp | Predicts fitness under cisplatin stress. This tests whether representations capture survival signals under drug-induced stress. |
| `GenerTeam/prokaryotic-gener-tasks::fitness_perchlorate_stress` | Fitness Browser | Gene-fitness prediction | Regression | 14.2k | 9:1 | avg 959 bp | Predicts fitness under perchlorate stress. This tests robustness-related signals in extreme chemical environments. |
| **Local sequence understanding: external zero-shot task** |  |  |  |  |  |  |  |
| `Marks-lab/RNAGym::RNAfitness` (13 prokaryote-related assays) | RNAGym / DMS assays | RNAfitness zero-shot evaluation | Zero-shot ranking / correlation evaluation | 54,384 | External benchmark; no training | assay-dependent | Tests zero-shot ranking of prokaryote-related RNA mutant fitness effects. The subset retains 13 assays whose `dms_id` or description contains prokaryote-related keywords. |
| **Regional and system-level long-context understanding: classification tasks** |  |  |  |  |  |  |  |
| `bgc_type_annotation_mibig4` | MIBiG 4.0 | BGC type annotation | Multilabel classification | 2,636 | 8:1:1 | 8 kb-1 Mb+ | Tests whether the model recognizes combinations of biosynthetic gene-cluster classes. |
| `bgc_vs_nonbgc_mibig4_gtdb226` | MIBiG / GTDB | BGC region detection | Binary classification | 4,214 | 8:1:1 | 8 kb-1 Mb+ | Tests whether the model distinguishes BGC regions from non-BGC genomic regions. |
| **Whole-genome embedding representation: classification task** |  |  |  |  |  |  |  |
| `macwiatrak/bacbench-phenotypic-traits-dna` | GIDEON | Strain phenotype prediction | Binary classification | 1,175 | Genus-level 7:1:2 | avg 3.7 Mb | Uses whole-genome embeddings to predict discrete strain phenotypes. This tests whether global genome representations retain signals related to physiology, lifestyle, and potential pathogenicity. |

---

## Evaluation Protocols and Results

### Unified Supervised Evaluation Protocol

- **Frozen backbone.** For all supervised tasks, the pretrained backbone is frozen. Only a downstream classification or regression head is trained.
- **Embedding extraction.** Sequences are passed through the model to extract representations from a specified layer. Unless otherwise stated, the final-layer representation is used and mean pooling is applied to obtain a sequence-level embedding.
- **Unified downstream evaluator.** Lightweight MLP heads are trained under the same data splits and task metrics to compare different representations.
- **Reported metrics.** Classification tasks use AUROC as the main metric and also report ACC and F1. Regression tasks use Pearson correlation as the main metric.

### Zero-Shot RNAfitness Evaluation Protocol

- **Task definition.** RNAGym RNAfitness is a deep mutational scanning benchmark for RNA variant-effect prediction. The goal is to evaluate agreement between model scores and experimental `DMS_score` or fitness readouts.
- **Sequence scoring.** Because Genos-m is a DNA causal language model, RNA sequences are converted from U to T before scoring. Whole-sequence next-token average log-likelihood is used as the model score for each mutant. No supervised training is performed.
- **Result aggregation.** Spearman correlation between model scores and experimental readouts is computed within each assay. Following the direction-independent RNAGym convention, absolute Spearman is used and then averaged across the selected 13 prokaryote-related assays. Spearman, AUC, and MCC are reported.

---

## 1. Local Sequence Understanding

### 1.1 Essential gene identification

**Task description.** This task predicts whether a single gene is essential for basic bacterial survival. It evaluates whether the model captures sequence signals associated with fundamental cellular function and gene essentiality.

**Data source and processing.**

- The task uses the BacBench essential gene dataset (`macwiatrak/bacbench-essential-genes-dna`) [1]. Essentiality labels come from DEG, and BacBench maps DNA and protein sequences from GenBank using the RefSeq genome IDs provided by DEG.
- BacBench performs quality control on the original 66 genomes and removes redundant genomes with highly overlapping annotations. The quality-controlled dataset contains 51 genomes, 37 species, 22,486 essential genes, and 146,922 non-essential genes [1].

**Evaluation workflow.**

- The benchmark follows the BacBench train/validation/test split of 60%/20%/20%. Genomes from the same genus appear in only one split. This design reduces performance inflation caused by closely related lineages and evaluates generalization to held-out genera [1].
- MLP heads are trained on 30 genomes, tuned on 10 validation genomes, and evaluated on the remaining 11 test genomes.
- The seed loop is repeated three times, and mean performance and variation are reported.

**Metrics.** AUROC, ACC, and F1.

---

### 1.2 Promoter recognition

**Task description.** This task predicts whether a short sequence from a given species background is a prokaryotic promoter fragment. It evaluates whether the model represents local regulatory sequence grammar in an in-species sequence context.

**Data source and processing.**

- This task uses the prokaryotic promoter benchmark released with iPro-MP [3]. The original iPro-MP study used experimentally validated promoters from the Prokaryotic Promoter Database. It retained 81-bp promoter windows from 23 prokaryotic species, defined as the -60 to +20 region relative to the transcription start site. After CD-HIT deduplication, 107,286 positive examples remained.
- Negative examples were constructed by the iPro-MP team from CDS regions and convergent intergenic regions of the corresponding species genomes. After CD-HIT deduplication, negatives were randomly sampled to produce a 1:2 positive-to-negative ratio [3].
- This benchmark follows the task definition and train/test split released by iPro-MP, which uses an 8:2 random split [3].

**Evaluation workflow.**

- The unified supervised evaluation protocol is used.
- Following iPro-MP, species-specific modelling is used. Independent classifiers are trained for the 23 species.

**Metrics.** AUROC, ACC, and F1.

---

### 1.3 Six-class bacterial gene classification

**Task description.** This task classifies DNA fragments into six functional region types: coding sequence (CDS), pseudogene, transfer RNA (tRNA), ribosomal RNA (rRNA), non-coding RNA (ncRNA), and intergenic region. It evaluates whether the model distinguishes sequence features of different genomic functional regions. Distinguishing CDS from pseudogene is particularly challenging because pseudogenes are often highly similar to functional coding regions but have lost their original function through disruptive mutations or rearrangements.

**Data source and processing.** This task reproduces the GenerAnno `gene_classification_bacteria` dataset [2]. The data are a six-class collection constructed from RefSeq annotations. The current dataset contains about 30,000 DNA fragments, is approximately balanced across the six classes, and has an average length of 837 bp.

**Evaluation workflow.** The benchmark follows the GenerAnno evaluation setting and uses 10-fold cross-validation. In each round, one fold is used as the test set and the remaining samples are used for training and validation. Model representations and downstream heads follow the unified supervised protocol [2].

**Metrics.** AUROC, ACC, and F1.

---

### 1.4 Antibiotic Resistance Gene Prediction

**Task description.** This task predicts whether a single gene sequence is associated with antibiotic resistance. It follows the GenerAnno `drug_resistence_prediction` setting and is a gene-level binary classification task. The goal is to evaluate whether the model recognizes resistance-related functional signals from a single gene sequence.

**Data source and processing.**

- This task reproduces the GenerAnno `drug_resistence_prediction` dataset [2].
- In the original task, positive examples are resistance-related genes from CARD. Negative examples are non-AMR control genes sampled from RefSeq, with length-distribution matching to reduce non-biological confounding [2]. The task does not evaluate strain-level antibiotic-resistance phenotypes.
- The current benchmark contains about 16.1k samples, with about 14.5k training samples and 1.6k test samples under an 8:2 train/test split.

**Evaluation workflow.**

- The unified supervised evaluation protocol is used.
- Binary classifiers are trained and tested according to the task labels.

**Metrics.** AUROC, ACC, and F1.

---

### 1.5 Virulence Factor Gene Identification

**Task description.** This task predicts whether a single gene sequence is a virulence factor gene. It evaluates whether gene-level representations capture functional signals associated with pathogenicity. Here, virulence factors refer mainly to functional genes involved in pathogenic processes, including toxins, adhesion, invasion, immune evasion, iron uptake systems, secretion systems, and related effector proteins.

**Data source and processing.**

- This study curates a gene-level binary classification dataset from VFDB and a non-virulence control set.
- Positive examples come from representative virulence-associated genes in the VFDB core dataset. Sequences longer than 8 kb are removed, leaving 4,570 positive examples [6].
- Negative examples are derived from DEG essential genes [1].
- During negative-set construction, entries with annotations or descriptions suggesting possible virulence relevance are removed. These include classical toxins or virulence factors, secretion systems and effectors, capsules, polysaccharides, and outer-structure genes. Stratified random sampling by species proportion is then applied, yielding 4,989 non-virulence essential genes as negative controls.

**Evaluation workflow.** The final dataset is split into train/test partitions at 8:2 using label stratification. Model representations and downstream heads follow the unified supervised evaluation protocol.

**Metrics.** AUROC, ACC, and F1.

### Local Sequence Classification Results

Genos-m ranks among the top two models across all five local sequence classification tasks. In antibiotic-resistance gene detection, it achieves the highest AUROC/AUC, ACC, and F1 scores: 0.9896, 0.9532, and 0.9531. Overall, these results show that, with a frozen backbone and a lightweight downstream head, Genos-m local sequence representations can support diverse functional discrimination tasks. Full AUC, ACC, and F1 results for the MLP classification tasks are provided in Table S1.

**Table 2. Local sequence classification performance (AUROC)**

| Task | DNABERT-2 | Nucleotide Transformer v2 | Nucleotide Transformer v3 | GenerAnno | Evo1-7B | Evo2-40B | Genos-m |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Promoter recognition | 0.6820 | 0.7773 | 0.7989 | 0.7804 | 0.7185 | **0.8504** | <ins>0.8460</ins> |
| Antibiotic-resistance gene detection | 0.9176 | 0.9727 | 0.9529 | 0.9855 | 0.9313 | <ins>0.9884</ins> | **0.9896** |
| Virulence-factor detection | 0.7840 | 0.8942 | 0.7939 | 0.9301 | 0.8161 | **0.9617** | <ins>0.9548</ins> |
| Essential gene identification | 0.6658 | 0.6921 | 0.7489 | 0.7953 | 0.7253 | **0.8619** | <ins>0.8534</ins> |
| Six-class bacterial gene classification | 0.9637 | 0.9866 | 0.9848 | 0.9884 | 0.9711 | **0.9937** | <ins>0.9932</ins> |

---

### 1.6 Gene Fitness Prediction

**Task description.** This task predicts gene-fitness scores under specific nutrient or chemical-stress conditions from gene sequences. It evaluates whether the model extracts condition-dependent adaptation signals from gene sequences. In Prokaryotic Gener Tasks, gene-fitness is defined as the contribution of a gene to microbial survival and adaptation under a specific experimental condition. The task therefore concerns condition-dependent gene importance, not general functional prediction across all environments.

**Data source and processing.**

- This task uses condition-specific gene-fitness data from Fitness Browser [2] and follows the GenerAnno gene-fitness prediction setting.
- The current benchmark selects eight representative subtasks covering nutrient utilization and chemical stress: minimal media glucose, L-arabinose C, pyruvate C, D-alanine N, ammonium chloride N, L-histidine nutrient, cisplatin stress, and perchlorate stress.

**Table 3. Environmental categories and biological meaning of gene-fitness subtasks**

| Environmental category | Examples | Biological meaning |
| --- | --- | --- |
| Nutrient variation | Glucose as a basal carbon source, L-histidine as a nutrient, pyruvate as an alternative carbon source, L-arabinose as a specific carbon source, D-alanine, and ammonium chloride as nitrogen sources | Evaluates whether the model captures gene-nutrient metabolism associations and distinguishes important metabolic genes under different carbon and nitrogen sources. |
| Chemical stress | Cisplatin as drug toxicity and perchlorate as an environmental contaminant | Evaluates whether the model predicts toxicity-resistance and stress-response genes under chemical stress. |

**Evaluation workflow.**

- The unified supervised evaluation protocol is used.
- MSE is used as the loss.

**Metric.** Pearson correlation.

### Results

Across the eight gene-fitness regression subtasks, Genos-m achieves the best performance among the compared models on five subtasks and ranks second to Evo2-40B on the remaining three. These results indicate that Genos-m gene-level sequence representations support condition-specific gene-fitness prediction and remain stable across nutrient and chemical-stress conditions. Task-level Pearson correlations are shown in Table 4.

**Table 4. Gene-fitness prediction performance under different environmental conditions (Pearson correlation)**

| Task | DNABERT-2 (117M) | Nucleotide Transformer v2 (500M) | Nucleotide Transformer v3 (650M) | GenerAnno (500M) | Evo1-7B | Evo2-40B | Genos-m |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Ammonium chloride N | 0.1196 | 0.3231 | 0.2981 | 0.4671 | 0.2386 | **0.5157** | <ins>0.5154</ins> |
| Cisplatin stress | 0.1253 | 0.1708 | 0.1308 | 0.2048 | 0.1232 | **0.3381** | <ins>0.3293</ins> |
| D-Alanine N | 0.0919 | 0.2394 | 0.2354 | 0.4101 | 0.2358 | **0.4734** | <ins>0.4314</ins> |
| L-Arabinose C | 0.1869 | 0.3654 | 0.3147 | 0.4807 | 0.2071 | <ins>0.5407</ins> | **0.5914** |
| L-Histidine nutrient | 0.1384 | 0.5440 | 0.3095 | 0.6340 | 0.2500 | <ins>0.6685</ins> | **0.7479** |
| Min-media glucose | 0.1718 | 0.5422 | 0.2988 | 0.6605 | 0.2578 | <ins>0.6936</ins> | **0.7513** |
| Pyruvate C | 0.1220 | 0.2805 | 0.2652 | <ins>0.4803</ins> | 0.1662 | 0.4661 | **0.5495** |
| Perchlorate stress | 0.1263 | 0.1695 | 0.0502 | 0.1494 | 0.0530 | <ins>0.2727</ins> | **0.2981** |

---

### 1.7 RNAfitness Prediction, Zero-Shot

**Task description.** This task uses the RNAGym `RNAfitness` subset to evaluate whether Genos-m can rank RNA sequence variant effects without task-specific fine-tuning or downstream-head training. Given an RNA mutant sequence, the model directly assigns a language-model score. Prediction quality is assessed by correlation with RNAGym experimental `DMS_score` or fitness readouts. This external zero-shot benchmark tests whether Genos-m scores agree with experimental variant-effect measurements in prokaryote-related RNA assays.

**Data source and processing.**

- This task uses RNAGym RNAfitness data and selects 13 prokaryote-related assays [4]: Andreasson_2020_ribozyme, BCHB_CHLTE_Tsuboyama_2023_2KRU, BLAT_ECOLX_Firnberg_2014, BLAT_ECOLX_Jacquier_2013, CCDB_ECOLI_Adkar_2012, ESTA_BACSU_Nutschel_2020, F7YBW8_MESOW_Ding_2023, IF1_ECOLI_Kelsic_2016, MLAC_ECOLI_MacRae_2023, PSAE_PICP2_Tsuboyama_2023_1PSE, Peri_2022_ribozyme, Q837P4_ENTFA_Meier_2023, and RNC_ECOLI_Weeks_2023.
- Assays are selected when their `dms_id` or description contains prokaryote-related keywords. The current subset contains 54,384 mutant sequences.

**Evaluation workflow.**

- The RNAfitness zero-shot protocol is used. No task-specific fine-tuning is performed, and no downstream prediction head is trained.
- Other comparison-model results are taken from the RNAGym full-benchmark release and recomputed as assay-level Spearman means over the same 13 assays [4].

**Metrics.** Spearman correlation, AUROC, and MCC.

### Results

On the 13 prokaryote-related RNAfitness assays, Genos-m achieves a high average Spearman correlation and ranks second among the public comparison models, after Evo2-7B. This result indicates that, without task-specific training, Genos-m provides useful ranking signals for some prokaryote-related RNA mutant DMS and fitness readouts.

**Table 5. Zero-shot performance of Genos-m and comparison models on the prokaryote-related RNAGym RNAfitness subset**

| Model | Spearman | AUC | MCC |
| --- | --- | --- | --- |
| **Evo2-7B** | **0.390** | **0.691** | **0.284** |
| Genos-m | <ins>0.313</ins> | <ins>0.653</ins> | <ins>0.224</ins> |
| Evo1.5 | 0.289 | 0.640 | 0.208 |
| Evo1 | 0.264 | 0.626 | 0.179 |
| RNAErnie | 0.244 | 0.617 | 0.174 |
| Nucl. Transformer | 0.178 | 0.579 | 0.105 |
| RNA-FM | 0.168 | 0.577 | 0.103 |
| RiNALMo | 0.141 | 0.561 | 0.085 |
| GenSLM | 0.069 | 0.530 | 0.041 |

---

## 2. Regional and System-Level Long-Context Evaluation

Regional and system-level long-context tasks focus on biosynthetic gene cluster (BGC) sequences. Unlike single-gene tasks, BGCs typically consist of multiple neighboring genes. The model must integrate gene colocalization, module boundaries, and combinatorial functional information across longer contexts.

### 2.1 BGC Binary Classification

**Task description.** Given a long genomic region, this task predicts whether the region is a biosynthetic gene cluster. It evaluates whether the model identifies combinatorial functional signals associated with BGCs in multi-gene regional contexts.

**BGC definition.** A biosynthetic gene cluster is a tightly linked group of genes on a chromosome or plasmid. These genes usually encode enzymes, regulators, and transporters that jointly support the biosynthesis, regulation, and export of natural products, also known as secondary metabolites.

**Research relevance.** Natural products from BGCs are important sources of antibiotics, anticancer drugs, and immunosuppressants. Some BGCs are also related to bioremediation, such as pollutant degradation, while antimicrobial peptides may provide alternative routes for antibiotic development.

**Data source and processing.**

- Positive examples are experimentally validated MIBiG v4.0 BGCs. After filtering fungal, unknown, and abnormal-length entries, 2,107 examples remain [7].
- Negative examples are non-BGC regions. antiSMASH is used to identify and exclude regions containing BGCs, and non-BGC fragments are then sampled from BGC-free contigs. Sampling matches the length and CDS-count distributions of positive examples to reduce confounding from sequence length or gene density. The final negative set contains 2,107 non-BGC regions, giving a 1:1 positive-to-negative ratio.
- The current dataset is split into train/validation/test sets at 8:1:1 while preserving class balance.

**Evaluation workflow.**

- The unified supervised evaluation protocol is used.
- Inputs are processed according to each model's maximum context length. For sequences exceeding a model context window, truncation or sliding-window aggregation may be used.

**Metrics.** AUROC, ACC, and F1.

### 2.2 BGC Multilabel Classification

**Task description.** Given a known BGC region sequence, this task predicts its possible biosynthetic classes. Because a single BGC may contain multiple biosynthetic features, such as both PKS and NRPS modules, the task is defined as multilabel classification rather than mutually exclusive six-class classification. For each BGC, the model predicts one or more labels among PKS, NRPS, ribosomal/RiPP, saccharide, terpene, and other.

**Data source and processing.** This benchmark curates BGC regional sequences and biosynthetic class annotations from MIBiG v4.0 [7]. Labels are merged into six classes: PKS, NRPS, ribosomal/RiPP, saccharide, terpene, and other. Each BGC sequence may have one or more class labels, so the sum of positive examples across labels may exceed the number of BGC samples. This task evaluates whether the model extracts multi-gene organization features associated with different biosynthetic classes.

**Evaluation workflow.**

- The dataset is split into train/validation/test sets at 8:1:1 while preserving the distribution of each label as much as possible.
- The unified supervised evaluation protocol is used. Because class distribution is imbalanced, focal loss is used for classifier training.
- The model outputs six label probabilities for each BGC. For each label, an optimal threshold is independently selected on the validation set and then used to convert probabilities into multilabel predictions.

**Metrics.** AUROC, ACC, and F1.

### BGC Sequence Classification Results

Genos-m performs consistently on both regional BGC tasks. For BGC versus non-BGC region detection, Genos-m reaches AUROC = 0.9907, close to Evo2-40B at 0.9911 and higher than Nucleotide Transformer v3 at 0.9365 and GenerAnno at 0.9136. This indicates that Genos-m supports BGC presence classification for given candidate regions. For BGC multilabel classification type annotation, Genos-m reaches AUROC = 0.9216, the best value among the compared models. This suggests that its representations capture multi-gene organization features associated with known BGC biosynthetic classes. Full AUC, ACC, and F1 values for the two BGC MLP tasks are provided in Table S1.

**Table 6. BGC regional sequence classification performance (AUROC)**

| Task | DNABERT-2 | Nucleotide Transformer v2 | Nucleotide Transformer v3 | GenerAnno | Evo1-7B | Evo2-40B | Genos-m |
| --- | --- | --- | --- | --- | --- | --- | --- |
| BGC binary classification | 0.8551 | 0.9049 | 0.9365 | 0.9136 | 0.7984 | **0.9911** | <ins>0.9907</ins> |
| BGC multilabel classification | 0.7897 | 0.7845 | 0.8271 | 0.8207 | 0.8101 | <ins>0.8951</ins> | **0.9216** |

---

## 3. Whole-Genome Understanding

### Genomic Phenotype Prediction

**Task description.** This task predicts strain phenotypes from whole-genome embeddings. It evaluates whether the model extracts global information from complete genome sequences that is relevant to discrete bacterial phenotypes.

Given a genome embedding, an independent classifier is trained for each phenotype to predict whether the genome has that phenotype.

**Data source and processing.**

- The phenotype data used in this benchmark come from the GIDEON database and contain 1,175 genomes and five discrete classification phenotypes [1].
- Data are split at the genus level into train/validation/test sets at 0.7/0.1/0.2 and evaluated under multiple random seeds.
- Seeds are [1, 2, 3].

**Table 7. Binary strain phenotype labels and sample counts for whole-genome representation evaluation**

| Phenotype | Group | Label | Samples | Description | Source |
| --- | --- | --- | --- | --- | --- |
| `gideon_Gram positive` | +/- | positive / negative | 550 / 602 | Gram staining status | GIDEON |
| `gideon_Anaerobe` | +/- | anaerobe / non-anaerobe | 322 / 789 | Oxygen tolerance | GIDEON |
| `gideon_Motile` | +/- | motile / non-motile | 292 / 746 | Motility | GIDEON |
| `gideon_Spore formation` | +/- | spore-forming / non-spore-forming | 95 / 1074 | Spore formation | GIDEON |
| `gideon_Beta hemolysis` | +/- | beta-hemolysis positive / negative | 52 / 631 | Beta-hemolytic activity | GIDEON |

**Evaluation workflow.**

- Linear probing is used. Genos-m first splits each bacterial genome into 1 Mbp chunks, produces chunk-level embeddings, and averages all chunk embeddings to obtain a genome-level embedding. A separate linear classifier is then trained for each phenotype.
- Mean performance and variation are evaluated under seeds [1, 2, 3].

**Metric.** AUROC.

### Results Overview

Using genome-level embeddings, Genos-m performs close to Bacformer, a model designed for genome-scale modelling. This suggests that its 1 Mbp chunk-aggregated representations retain some global sequence signals related to discrete strain phenotypes.

**Table 8. Performance comparison on bacterial phenotype prediction (AUROC, mean +/- SD)**

| Phenotype/Model | Mistral-DNA | DNABERT-2 | Nucleotide Transformer | ProkBERT | ESM-2 | ESM-C | ProtBERT | gLM2 | Bacformer | Genos-m |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| gideon Gram positive | 65.58 +/- 6.77 | 86.88 +/- 5.47 | 88.12 +/- 8.21 | 97.07 +/- 3.41 | <ins>98.48 +/- 1.69</ins> | **99.73 +/- 0.29** | 97.50 +/- 2.39 | 96.06 +/- 4.29 | 97.61 +/- 3.16 | 98.41 +/- 1.77 |
| gideon Anaerobe | 77.21 +/- 2.99 | 88.84 +/- 4.07 | 90.90 +/- 1.02 | 92.50 +/- 1.52 | 96.14 +/- 1.34 | 96.57 +/- 2.93 | 97.92 +/- 1.07 | 91.08 +/- 1.84 | **98.92 +/- 0.37** | <ins>98.14 +/- 0.57</ins> |
| gideon Motile | 66.98 +/- 9.54 | 76.09 +/- 2.49 | 81.63 +/- 3.84 | 64.59 +/- 5.22 | 82.07 +/- 4.29 | 74.70 +/- 8.69 | 79.50 +/- 6.52 | 63.98 +/- 5.82 | **87.01 +/- 9.71** | <ins>83.22 +/- 3.52</ins> |
| gideon Spore formation | 52.59 +/- 21.62 | 72.71 +/- 16.94 | 79.25 +/- 27.78 | 83.66 +/- 22.84 | 89.55 +/- 8.27 | 89.91 +/- 12.13 | <ins>90.68 +/- 7.52</ins> | 74.53 +/- 30.76 | **90.85 +/- 10.38** | 90.24 +/- 14.58 |
| gideon Beta hemolysis | **63.40 +/- 15.20** | <ins>62.26 +/- 16.78</ins> | 60.54 +/- 16.54 | 53.06 +/- 5.76 | 54.80 +/- 10.50 | 54.08 +/- 6.86 | 53.71 +/- 12.93 | 50.10 +/- 5.57 | 52.05 +/- 8.24 | 55.50 +/- 1.59 |

## Summary and Conclusions

Except for the fitness tasks, classification results are reported with AUC. Fitness reports the mean Pearson correlation across eight gene-fitness regression subtasks. Full MLP AUC, ACC, and F1 values for classification tasks are provided in Table S1, and task-level fitness Pearson correlations are provided in Table 4.

**Genos-m Benchmarks evaluates whether a single pretrained representation is useful across microbial genome tasks at different sequence scales.** Under a frozen-backbone setting with only lightweight downstream models trained, Genos-m representations support short-sequence and gene-level functional discrimination, condition-specific gene-fitness regression, BGC region detection and type classification, and whole-genome strain phenotype prediction. In RNAfitness zero-shot evaluation, the model can rank experimental readouts for some prokaryote-related RNA mutants without task-specific training. These results support Genos-m as an effective microbial genome representation model, but the conclusion is bounded by the current task definitions, data splits, and evaluation protocols. It should not be extrapolated to a universal functional annotation system, a general RNA foundation model, or a general strain phenotype predictor in open environments.

In local sequence tasks, Genos-m ranks among the top models for promoter recognition, essential gene identification, antibiotic-resistance gene detection, virulence-factor detection, and gene-type classification. This indicates that its local sequence representations support discrimination of regulatory elements, functional regions, and resistance- or virulence-related sequence signals. In the eight gene-fitness regression subtasks, Genos-m reaches the best performance on five tasks and ranks second to Evo2-40B on the other three. This further suggests that its gene-level representations provide effective features for continuous functional readouts under different nutrient and chemical-stress conditions.

In regional long-context tasks, Genos-m approaches the best model on BGC versus non-BGC classification and achieves the best performance on BGC multilabel classification type annotation. This indicates that the model can integrate multi-gene context from given regional sequences and represent sequence features related to BGC functional organization. This conclusion is limited to classification and annotation of given regions. It does not imply complete-genome BGC boundary detection capability.

In external RNAfitness zero-shot evaluation, Genos-m achieves mean Spearman = 0.313 across 13 prokaryote-related assays and ranks second among public comparison models, after Evo2-7B. This suggests that its model scores provide useful ranking signals for some prokaryote-related RNA mutant-effect assays.

In whole-genome embedding tasks, Genos-m splits each bacterial genome into 1 Mbp chunks, generates chunk-level embeddings, and averages them into a genome-level embedding. In genus-separated GIDEON/BacBench phenotype prediction, Genos-m performs close to Bacformer, a model designed for genome-scale modelling, and reaches the top tier for several phenotype labels. This indicates that its nucleotide-level chunk-aggregated representations retain global sequence signals related to basic strain phenotypes. The result should not be interpreted as evidence for general strain phenotype prediction in open environments.

Overall, the main strength of Genos-m is the stable performance of one pretrained representation across multiple sequence scales and task settings. Under frozen-backbone supervised evaluation, Genos-m supports gene-fragment, BGC-region, and whole-genome embedding tasks. In RNAfitness zero-shot evaluation, it also provides ranking signals for some prokaryote-related RNA mutant readouts. Together with its corpus design for human-associated microbial genomes, million-base context modelling, and smaller active parameter count per forward pass, Genos-m can serve as a foundational representation model for microbial genome tasks, supporting model reuse, task expansion, standardized evaluation, and downstream application development.

---

**Table 9. Comparison of evaluated model types, parameter scales, and input configurations**

| Model | Variant / Checkpoint | Objective | Params | dim | Max context | Input type |
| --- | --- | --- | --- | --- | --- | --- |
| **DNA models** |  |  |  |  |  |  |
| Mistral-DNA | Mistral-DNA-v1-138M-bacteria | Autoregressive | 138M | 768 | 512 | DNA |
| DNABERT-2 [8] | DNABERT-2-117M | Masked | 117M | 768 | 512 | DNA |
| Nucleotide Transformer v2 [9] | nt-v2-250m-multi-species | Masked | 250M | 768 | 2,048 | DNA |
| Nucleotide Transformer v3 [10] | InstaDeepAI/NTv3_650M_pre | Masked | 500M | 1536 | 1M | DNA |
| GenSLM [11] | genslm_2.5B_patric | Autoregressive | 2.5B | 2560 | 2048 | DNA |
| GenerAnno [2] | GENERanno-eukaryote-0.5b-base | Masked | 500M | 1024 | 8,192 | DNA |
| Evo1-7B [12] | evo-1-8k-base (1.1_fix) | Autoregressive | 6.5B | 4,096 | 8,192 | DNA |
| Evo1.5-7B [12] | evo-design/evo-1.5-8k-base | Autoregressive | 7B | 4,096 | 8,192 | DNA |
| Evo2-40B [13] | evo2_40b_base | Autoregressive | 40B | 4,096 | 8,192 | DNA |
| ProkBERT [14] | [neuralbioinfo/prokbert-mini-long](https://huggingface.co/neuralbioinfo/prokbert-mini-long) | Masked | 27M | 384 | 4096 | DNA |
| Genos-m | Genos-m(cpt32k) | Autoregressive | 330M (4.7B) | 1,024 | 1M | DNA |
| **Protein models** |  |  |  |  |  |  |
| ESM-2 [15] | esm2_t12_35M_UR50D | Masked | 35M | 480 | 1,024 | Single protein |
| ESM-C [16] | esmc_300m | Masked | 300M | 960 | 1,024 | Single protein |
| ProtBERT [17] | Rostlab/prot_bert | Masked | 420M | 1,024 | 2048 | Single protein |
| Bacformer [5] | bacformer-masked-complete-genomes | Masked | 27M | 480 | 6,000 | Multiple protein |
| **RNA models** |  |  |  |  |  |  |
| RNAErnie [18] | RNAErnie | Masked | 86M | 768 | 512 | RNA |
| RiNALMo [19] | giga-v1 | Masked | 651M | 1280 | 1024 | RNA |
| RNA-FM [20] | rna_fm_t12 | Masked | 100M | 640 | 1024 | RNA |

**Table S1. MLP classification performance (AUC, ACC, and F1)**

| Task | Metric | DNABERT-2 (117M) | Nucleotide Transformer v2 (500M) | Nucleotide Transformer v3 (650M) | GenerAnno (500M) | Evo1-7B | Evo2-40B | Genos-m |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Essential gene identification | AUC | 0.6658 | 0.6921 | 0.7489 | 0.7953 | 0.7253 | **0.8619** | <ins>0.8534</ins> |
|  | ACC | 0.8704 | 0.8689 | 0.8726 | 0.8863 | 0.8678 | <ins>0.8919</ins> | **0.8981** |
|  | F1 | 0.5644 | 0.5241 | 0.5452 | 0.6733 | 0.5103 | <ins>0.7383</ins> | **0.7561** |
| Promoter recognition | AUC | 0.6820 | 0.7773 | 0.7989 | 0.7804 | 0.7185 | **0.8504** | <ins>0.8460</ins> |
|  | ACC | 0.6937 | 0.7471 | 0.7553 | 0.7502 | 0.7135 | **0.8011** | <ins>0.7970</ins> |
|  | F1 | 0.6021 | 0.6903 | 0.6990 | 0.6959 | 0.6210 | **0.7679** | <ins>0.7562</ins> |
| Six-class bacterial gene classification | AUC | 0.9637 | 0.9866 | 0.9848 | 0.9884 | 0.9711 | **0.9937** | <ins>0.9932</ins> |
|  | ACC | 0.7853 | 0.8887 | 0.8817 | 0.9053 | 0.8026 | **0.9322** | <ins>0.9280</ins> |
|  | F1 | 0.7821 | 0.8884 | 0.8814 | 0.9052 | 0.7980 | **0.9322** | <ins>0.9281</ins> |
| Antibiotic-resistance gene detection | AUC | 0.9176 | 0.9727 | 0.9529 | 0.9855 | 0.9313 | <ins>0.9884</ins> | **0.9896** |
|  | ACC | 0.8274 | 0.9103 | 0.8818 | 0.9440 | 0.8557 | <ins>0.9459</ins> | **0.9532** |
|  | F1 | 0.8272 | 0.9102 | 0.8817 | 0.9440 | 0.8557 | <ins>0.9459</ins> | **0.9531** |
| Virulence-factor detection | AUC | 0.7840 | 0.8942 | 0.7939 | 0.9301 | 0.8161 | **0.9617** | <ins>0.9548</ins> |
|  | ACC | 0.7123 | 0.8086 | 0.6878 | 0.8572 | 0.7385 | **0.8980** | <ins>0.8792</ins> |
|  | F1 | 0.7116 | 0.8079 | 0.6838 | 0.8568 | 0.7381 | **0.8977** | <ins>0.8787</ins> |
| BGC binary classification | AUC | 0.8551 | 0.9049 | 0.9365 | 0.9136 | 0.7984 | **0.9911** | <ins>0.9907</ins> |
|  | ACC | 0.7464 | 0.8081 | 0.8531 | 0.8389 | 0.7180 | <ins>0.9076</ins> | **0.9289** |
|  | F1 | 0.7358 | 0.8039 | 0.8510 | 0.8378 | 0.6893 | <ins>0.9069</ins> | **0.9287** |
| BGC multilabel classification | AUC | 0.7897 | 0.7845 | 0.8271 | 0.8207 | 0.8101 | <ins>0.8951</ins> | **0.9216** |
|  | ACC | 0.2679 | 0.2919 | 0.3206 | 0.3732 | 0.3062 | <ins>0.5550</ins> | **0.5742** |
|  | F1 | 0.5286 | 0.5242 | 0.5581 | 0.5862 | 0.5257 | <ins>0.6512</ins> | **0.7384** |

## References

[1] Wiatrak M, Weimann A, Vinas Torne R, et al. BacBench: Evaluating Genomic Language Models for Bacteria. OpenReview, ICLR 2026 Conference Withdrawn Submission, 2025 [accessed 2026-03-18].

[2] Li Q, Wu W, Zhu Y, et al. GENERanno: A Genomic Foundation Model for Metagenomic Annotation. bioRxiv, 2025. DOI: 10.1101/2025.06.04.656517.

[3] Su W, Yang Y, Zhao Y, et al. iPro-MP: a BERT-based model to predict multiple prokaryotic promoters. Genome Biology, 2025, 26(1):353. DOI: 10.1186/s13059-025-03819-9.

[4] Arora R, Angelo M, Choe C A, et al. RNAGym: Large-scale Benchmarks for RNA Fitness and Structure Prediction. bioRxiv, 2025. DOI: 10.1101/2025.06.16.660049.

[5] Wiatrak M, Vinas Torne R, Ntemourtsidou M, et al. A contextualised protein language model reveals the functional syntax of bacterial evolution. bioRxiv, 2025. DOI: 10.1101/2025.07.20.665723.

[6] Zhou S, Liu B, Zheng D, et al. VFDB 2025: an integrated resource for exploring anti-virulence compounds. Nucleic Acids Research, 2025, 53(D1):D871-D877.

[7] Zdouc M M, Blin K, Louwen N L L, et al. MIBiG 4.0: advancing biosynthetic gene cluster curation through global collaboration. Nucleic Acids Research, 2025, 53(D1):D678-D690.

[8] Zhou Z, Ji Y, Li W, et al. DNABERT-2: Efficient foundation model and benchmark for multi-species genome. arXiv preprint arXiv:2306.15006, 2023.

[9] Dalla-Torre H, Gonzalez L, Mendoza-Revilla J, et al. Nucleotide Transformer: building and evaluating robust foundation models for human genomics. Nature Methods, 2025, 22(2):287-297. DOI: 10.1038/s41592-024-02523-z.

[10] Boshar S, Evans B, Tang Z, et al. A foundational model for joint sequence-function multi-species modeling at scale for long-range genomic prediction. bioRxiv, 2025. DOI: 10.64898/2025.12.22.695963.

[11] Zvyagin M, Brace A, Hippe K, et al. GenSLMs: Genome-scale language models reveal SARS-CoV-2 evolutionary dynamics. The International Journal of High Performance Computing Applications, 2023, 37(6):683-705. DOI: 10.1177/10943420231201154.

[12] Nguyen E, Poli M, Durrant M G, et al. Sequence modeling and design from molecular to genome scale with Evo. Science, 2024. DOI: 10.1126/science.ado9336.

[13] Brixi G, Durrant M G, Ku J, et al. Genome modelling and design across all domains of life with Evo 2. Nature, 2026.

[14] Ligeti B, Szepesi-Nagy I, Bodnar B, et al. ProkBERT family: genomic language models for microbiome applications. Frontiers in Microbiology, 2024, 14:1331233. DOI: 10.3389/fmicb.2023.1331233.

[15] Lin Z, Akin H, Rao R, et al. Evolutionary-scale prediction of atomic-level protein structure with a language model. Science, 2023, 379(6637):1123-1130. DOI: 10.1126/science.ade2574.

[16] ESM Cambrian: Revealing the mysteries of proteins with unsupervised learning. EvolutionaryScale website, 2024.

[17] Elnaggar A, Heinzinger M, Dallago C, et al. ProtTrans: Towards Cracking the Language of Life's Code Through Self-Supervised Deep Learning and High Performance Computing. IEEE Transactions on Pattern Analysis and Machine Intelligence, 2021/2022. DOI: 10.1109/TPAMI.2021.3095381.

[18] Wang N, et al. Multi-purpose RNA language modelling with motif-aware pretraining and type-guided fine-tuning. Nature Machine Intelligence, 2024. DOI: 10.1038/s42256-024-00836-4.

[19] Penic R J, Vlasic T, Huber R G, et al. RiNALMo: general-purpose RNA language models can generalize well on structure prediction tasks. Nature Communications, 2025, 16:5671. DOI: 10.1038/s41467-025-60872-5.

[20] Chen J, Hu Z, Sun S, et al. Interpretable RNA Foundation Model from Unannotated Data for Highly Accurate RNA Structure and Function Predictions. arXiv / bioRxiv, 2022.
