# Genos-m 模型性能评测基准 (Genos-m Benchmarks)

Genos-m Benchmarks 是为评估 Genos-m 而组织的**原核微生物基因组基础模型评测集合**。该集合覆盖从短调控元件、基因级序列和区域级长上下文到全基因组表征的多尺度任务，用于衡量模型在**局部序列理解、区域/系统级长上下文建模及全基因组嵌入表征**中的下游可用性。该集合还纳入 RNAfitness 零样本任务，用于评估 DNA 预训练模型在外部 RNA 变异效应评测中的排序信号。

本评测集合中的局部序列、基因级、RNAfitness 和全基因组表型相关任务主要采用或改编自公开评测资源，包括 BacBench、GenerAnno、RNAGym 及 MIBiG 等相关公开数据库。

## 评测概述

Genos-m Benchmarks 将模型能力划分为三大核心评测维度，以实现从短片段序列表征到全基因组嵌入评估的系统性覆盖。

**1. 局部序列与基因级表征能力 (Local Sequence and Gene-Level Understanding)**

该类任务聚焦于短片段、基因及功能元件级序列，评估模型对局部生物学特征、调控信号、功能类别和环境适应性相关序列模式的表征能力。

*   必需基因识别
    
*   启动子识别
    
*   细菌基因六分类
    
*   抗生素抗性基因识别
    
*   毒力因子识别
    
*   基因适应性预测
    
*   RNAfitness 零样本评估
    

**2. 区域 / 系统级长上下文理解能力 (Regional / System-Level Long-Context Understanding)**

该类任务聚焦于多基因模块和长片段区域，评估模型在较长基因组上下文中识别结构化功能区域、整合邻近基因信息以及捕获区域级功能组织模式的能力。

*   BGC 二分类
    
*   BGC 多标签分类
    

**3. 全基因组理解能力 (Whole-Genome Understanding)**

该类任务基于 whole-genome embeddings，评估模型是否能够从整基因组序列中恢复谱系结构，并支持基因组水平的表型推断。

*   菌株表型预测
    

## 主要特性

*   **多维度生物学任务覆盖**：Genos-m Benchmarks 覆盖调控元件识别、基因区域/功能分类、BGC 区域识别与类型标注、特定条件下的基因适应性回归及菌株表型预测等任务，用于评估**预训练表示中是否保留了与功能、区域结构和表型相关的可用信号**。
    
*   **多尺度序列表征评估**：覆盖从几十 bp 的短调控元件、基因级序列、区域级长上下文到全基因组序列的表征-任务评估，用于系统衡量模型在不同序列尺度上的适用性。
    
*   **标准化下游评估流程**：使用轻量 MLP 下游分类器，并在统一数据划分和评估指标下比较不同表征的任务性能。分类任务的 MLP 结果以 AUC、ACC 和 F1 汇总于 Table S1。
    
*   **数据划分与泄漏控制**：不同任务按其原始 benchmark 设定采用属水平隔离、分层随机划分或 k-fold cross-validation。对具有谱系泄漏风险的任务，优先采用属水平或谱系隔离划分。
    

## 数据集与任务定义

> 评估目的：表 1 汇总本评测集合覆盖的任务、数据来源、划分方式和生物学含义。该表用于明确每个结果对应的任务定义，避免将单基因、区域级和全基因组任务混作同一类能力解读。

> 说明：表中 `序列长度` 如无特殊标注，指输入核酸片段的长度；对不同模型会按其 `max ctx` 对输入序列做截断（详见各任务的评测流程）。

**表 1. Genos-m Benchmarks 数据集清单与任务定义**

| **数据集** | **数据集来源** | **任务** | **任务类别** | **数据量** | **数据划分** | **序列长度** | **任务描述（生物学意义）** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **局部序列理解能力 (分类任务)** |  |  |  |  |  |  |  |
| `macwiatrak/bacbench-essential-genes-dna` | DEG / GenBank | 必需基因识别 | 二分类 | 169,408 | 按属切分 | avg 1k | **必需基因识别**：评估模型对必需基因序列特征的识别能力，通过按属切分严防谱系泄漏。 |
| `promoter_ipro_mp` | Prokaryotic Promoter Database | 启动子识别 | 二分类 | 321,858 | 8:2 | 81 bp | **启动子识别**：评估模型对短调控元件和局部序列语法的捕获能力，重点考察极短序列场景下的表征能力。 |
| `GenerTeam/prokaryotic-gener-tasks ：：gene_classification_bacteria` | RefSeq | 细菌基因六分类 | 六分类 | 30k | 9:1 | avg 837 | **基因分类**：评估模型区分不同基因组功能区域的能力，包括 CDS、pseudogene、tRNA、rRNA、ncRNA 和 intergenic region。 |
| `GenerTeam/prokaryotic-gener-tasks ：：drug_resistence_prediction` | CARD / RefSeq | 抗生素抗性基因识别 | 二分类 | 16.1k | 8:2 | 120–4.5k | **抗生素抗性基因识别**：评估模型识别抗性相关基因片段的能力；该任务为 gene-level binary classification。 |
| `virulence_gene_vfdb_coregenes` | VFDB / core genes negatives | 毒力因子识别 | 二分类 | 9,559 | 8:2 | < 8k | **毒力因子基因**：评估模型对毒力相关功能基因的识别能力，正样本为 VFDB 中经实验验证的代表性毒力相关基因；负样本为剔除功能注释为经典毒素/毒力因子、分泌系统/效应蛋白、荚膜、多糖及外层结构等潜在毒力相关基因后的DEG essential genes。 |
| **局部序列理解能力 (回归任务)**<br>评估目的：该部分基于 Fitness Browser 中的条件特异性基因适应性数据，评估模型能否从基因序列中提取与不同营养条件和化学胁迫相关的表征，并预测对应环境下的 gene fitness score。与二分类任务不同，该类任务以连续型适应性分数为输出，用于衡量模型在基因级环境适应性预测中的表征能力。 |  |  |  |  |  |  |  |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_Min-media-glucose` | Fitness Browser | 基因适应性预测 | 回归 | 35.6k | 9:1 | avg 1002 | **营养源(基础碳源葡萄糖)**：预测基因在葡萄糖最小培养基环境下的适应性分数，评估模型理解基础碳源代谢的能力。 |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_L-Arabinose_C` | Fitness Browser | 基因适应性预测 | 回归 | 60k | 9:1 | avg 1030 | **营养源 (阿拉伯糖)**：预测基因在以阿拉伯糖为唯一碳源条件下的适应性分数，评估模型对特殊碳源代谢途径的表征能力。 |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_Pyruvate_C` | Fitness Browser | 基因适应性预测 | 回归 | 60k | 9:1 | avg 1013 | **营养源 (丙酮酸)**：预测基因在丙酮酸作为碳源条件下的适应性分数，评估模型对能量代谢关键环节的建模能力。 |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_D-Alanine_N` | Fitness Browser | 基因适应性预测 | 回归 | 28.1k | 9:1 | avg 1015 | **营养源 (D-丙氨酸)**：预测基因在 D-丙氨酸作为氮源条件下的适应性分数，评估模型对特定氮源代谢的适应性建模。 |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_Ammonium-chloride_N` | Fitness Browser | 基因适应性预测 | 回归 | 37.6k | 9:1 | avg 1004 | **营养源 (氯化铵)**：预测基因在氯化铵作为氮源条件下的适应性分数，评估模型对氮源利用与氮代谢调控的预测能力。 |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_L-Histidine_nutrient` | Fitness Browser | 基因适应性预测 | 回归 | 35.6k | 9:1 | avg 1002 | **营养源 (L-组氨酸)**：预测基因在 L-组氨酸作为营养源条件下的适应性分数，评估模型对复杂营养调控的预测能力。 |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_Cisplatin_stress` | Fitness Browser | 基因适应性预测 | 回归 | 20.9k | 9:1 | avg 1028 | **化学压力 (顺铂)**：预测基因在顺铂（抗癌药物）胁迫下的适应性分数，评估模型对药物应激环境下关键生存基因的识别能力。 |
| `GenerTeam/prokaryotic-gener-tasks ：：fitness_perchlorate_stress` | Fitness Browser | 基因适应性预测 | 回归 | 14.2k | 9:1 | avg 959 | **化学压力 (高氯酸盐)**：预测基因在高氯酸盐化学胁迫下的适应性分数，评估模型在极端化学环境下的稳健性预测能力。 |
| **局部序列理解能力 (外部零样本任务)**<br>评估目的：该部分基于 RNAGym 中的 RNAfitness 数据，评估 Genos-m 在不进行任务特异性训练的情况下，对原核相关 RNA 序列变异效应的零样本排序能力。该任务采用 zero-shot 排序和 Spearman 相关性评估，用于衡量模型分数与实验 DMS / fitness readout 的一致性。 |  |  |  |  |  |  |  |
| `Marks-lab/RNAGym ：：RNAfitness (13 prokaryote-related assays subset)` | RNAGym / DMS assays | RNAfitness 零样本评估 | Zero-shot 排序 / 相关性评测 | 54,384 | 外部 benchmark；不训练 | 按 assay 变化 | **RNAfitness 零样本评估**：评估 Genos-m 对原核 RNA 序列变异适应性效应的零样本排序能力；该子集仅保留 dms_id 或 description 中包含原核相关关键词的 13 个 assays。 |
| **区域 / 系统级长上下文理解能力 (分类任务)**<br>评估目的：该部分聚焦于生物合成基因簇（biosynthetic gene clusters, BGCs）相关区域序列。评测任务不涉及从完整基因组中进行 BGC 边界定位，而是在给定区域序列上进行 BGC / non-BGC 判别和 BGC 类型多标签标注，用于评估模型对多基因区域上下文信息及功能组织模式的表征能力。 |  |  |  |  |  |  |  |
| `bgc_type_annotation_mibig4` | MIBiG 4.0 | BGC 类型标注 | 多标签分类 | 2,636 | 8:1:1 | 8K-1M+ | **类型标注**：评估模型对生物合成基因簇类型组合的识别能力。 |
| `bgc_vs_nonbgc_mibig4_gtdb226` | MIBiG / GTDB | BGC 区域识别 | 二分类 | 4,214 | 8:1:1 | 8K-1M+ | **区域识别**：评估模型从基因组序列中准确区分 BGC 区域与非 BGC 区域的能力。 |
| **全基因组嵌入表征能力 (分类任务)**<br>评估目的：该部分基于 whole-genome embeddings 进行菌株表型预测，评估模型是否能够从完整基因组序列中提取与菌株层面离散表型相关的全局表征。该任务采用按属切分的数据划分策略，以控制近缘谱系泄漏对评估的影响。 |  |  |  |  |  |  |  |
| `macwiatrak/bacbench-phenotypic-traits-dna` | Gideon | 菌株表型预测 | 二分类 | 1175 | 按属切分 7:1:2 | avg 3.7M | **菌株表型**：基于 whole-genome embedding 预测菌株离散表型，评估模型从整基因组序列抽取与生理性状、生活方式和潜在致病性相关全局信号的能力。 |

---

## 评测协议与结果

### 监督任务统一评测设置（Supervised evaluation protocol）

*   **Backbone 冻结**：对所有监督评测任务，默认冻结预训练模型，仅训练下游分类或回归头。
    
*   **Embedding 提取**：将序列输入模型，提取指定层的序列表征；默认使用最后一层表示，并采用 mean pooling 获得序列级 embedding。
    
*   **统一下游评估器**：在相同数据划分和评估指标下，使用轻量 MLP 作为下游分类或回归头评估模型性能。
    
*   **报告指标**：分类任务以 AUROC 为主指标，并报告 ACC 和 F1；回归任务以 Pearson 相关系数为主指标。
    

### RNAfitness 零样本评测设置（Zero-shot RNAfitness evaluation）

*   **任务性质**：RNAGym 的 RNAfitness 是基于 deep mutational scanning（DMS）assays 的 RNA variant-effect prediction benchmark，目标是评估模型分数与实验 `DMS_score` / fitness readout 之间的一致性。
    
*   **序列评分**：由于 Genos-m 是 DNA causal LM，先将 RNA 序列统一进行 U→T 转换后输入模型；对整条序列计算 next-token average log-likelihood，作为每个 mutant 的模型分数，不进行有监督的训练。
    
*   **结果汇总**：在单个 assay 内计算模型分数与实验 `DMS_score` / fitness readout 之间的 Spearman correlation；按 RNAGym 官方脚本的方向无关口径取 absolute Spearman，并在选定的 13 个原核相关 assays 上取 assay-level mean。本文报告 Spearman, AUC等指标。
    

---

### 1. 局部序列理解能力测评 (Local Sequence Understanding)

### 1.1 必需基因预测（Essential Gene Prediction）

**任务描述**   判断单个基因是否为维持细菌基本生存所必需的基因（essential gene vs. non-essential gene）。该任务用于评估模型在基因层面是否能够捕获与细胞基础功能和生存必需性相关的序列信号。

**数据来源与处理**

*   **数据来源**：采用 BacBench 中的 essential gene 数据集（`macwiatrak/bacbench-essential-genes-dna`）\[1\]。该数据集的 essentiality 标签来源于 DEG（Database of Essential Genes），并由 BacBench 作者根据 DEG 提供的基因组 RefSeq ID 从 GenBank 获取对应的 DNA 与蛋白序列。
    
*   **数据构建**：BacBench 对原始 66 个基因组进行质量控制，并去除注释集合高度重叠的冗余基因组，以降低重复基因组带来的评估偏倚。质控后数据集包含 51 个基因组、37 个物种，共 22,486 个 essential genes 和 146,922 个 non-essential genes\[1\]。
    

**评测流程**

*   当前评测沿用 BacBench 的训练/验证/测试划分策略，按照 **train/val/test = 60%/20%/20%** 划分数据。同一属的基因组仅出现在同一个 split 中，不跨训练集、验证集和测试集。该设置用于减少近缘谱系泄漏对模型性能的高估，并评估模型在 held-out genera 上的任务泛化表现\[1\]。
    
*   使用 30 个基因组训练 MLP 分类头，10 个基因组用于验证调优，其余 11 个基因组用于测试。
    
*   seed loop 重复 3 次，报告均值和波动。
    

**评估指标**   AUROC、ACC 和 F1。

---

### 1.2 启动子预测（Promoter Prediction）

**任务描述**   判断同一物种背景下的给定短序列是否为原核启动子片段，即 promoter vs non-promoter。该任务用于评估模型在物种内序列背景中对局部调控序列语法的表征能力。

**数据来源与处理**

*   本任务使用 iPro-MP 文章发布的原核启动子识别 benchmark dataset \[3\]。iPro-MP 原文基于 Prokaryotic Promoter Database（PPD）中实验验证的启动子序列，筛选得到覆盖 23 个原核物种的 81 bp 启动子片段；序列窗口定义为相对于转录起始位点（TSS）的 −60,+20−60,+20 区间，并经 CD-HIT 去冗余后保留 107,286 条正样本。
    
*   负样本由 iPro-MP 团队从对应物种基因组的 CDS 区域和 convergent intergenic 区域中构建，并经 CD-HIT 去冗余后随机抽样，使正负样本比例为 1:2 \[3\] 。
    
*   当前 benchmark 沿用 iPro-MP 发布数据集的任务定义与训练/测试划分；其将数据集随机划分为训练集和独立测试集，比例为 8:2 \[3\]。
    

**评测流程**

*   采用通用监督评测设置。
    
*   按 iPro-MP 设定进行物种特异性（species-specific）建模，为 23 个物种分别训练独立分类器。
    

**评估指标**   AUROC、ACC 和 F1。

---

### 1.3 基因类型分类（Gene Classification）

**任务描述**   将 DNA 片段分类为 6 类功能区域：编码序列（CDS）、假基因（Pseudo）、转运 RNA（tRNA）基因、核糖体 RNA（rRNA）基因、非编码 RNA（ncRNA）、基因间区（Intergenic）。该任务用于评估模型对不同基因组功能区域序列特征的区分能力。 CDS 与 pseudogene 的区分具有挑战性，因为假基因通常与功能性编码区域高度相似，但因破坏性突变或重排丧失原有功能。

**数据来源与处理**   本任务基于 Generanno 的 `gene_classification_bacteria` 数据集进行复现\[2\] ，数据来自 RefSeq 注释序列构建的六分类集合。当前数据集包含约 30,000 条 DNA 片段，六类基本均衡，平均长度 837 bp。

**评测流程**   评测口径与 Generanno 文献保持一致，采用 10-fold cross-validation。每轮以 1 折作为测试集，其余样本用于训练与验证；模型表征和下游头采用通用监督评测设置\[2\]。

**评估指标**   AUROC、ACC 和 F1。

---

### 1.4 抗生素抗性基因识别（Antibiotic resistance gene Prediction）

**任务描述**   判断给定单基因序列是否为抗生素抗性相关基因（AMR gene vs. non-AMR gene）。该任务对齐 **Generanno** 的 `drug_resistence_prediction` 设定，属于 **gene-level binary classification**，用于评估模型在单基因序列层面对抗性相关功能信号的识别能力。

**数据来源与处理**

*   本任务基于 Generanno 的 `drug_resistence_prediction` 数据集进行复现\[2\] 。
    
*   源任务中，正样本为 CARD 中整理的抗性相关基因，负样本为 RefSeq 中抽取的非 AMR 对照基因，并通过长度分布匹配降低非生物学因素对分类结果的影响\[2\] 。该任务的核心目的不是预测菌株抗药表型，而是检验模型能否从单个基因序列中识别抗性相关功能信号。
    
*   当前 benchmark 使用的数据集规模约为 **16.1k**，训练集约 **14.5k**，测试集约 **1.6k**，按 8:2 进行训练/测试划分。
    

**评测流程**

*   采用通用监督评测设置。
    
*   按任务标签进行二分类训练和测试。
    

**评估指标**   AUROC、ACC 和 F1。

---

### 1.5 毒力因子识别（Virulence factor genes Identification）

**任务描述**   判断给定单基因序列是否为毒力因子基因（VF gene vs. non-VF gene）。该任务用于评估模型在单基因序列层面对毒力相关功能信号的识别能力。这里的毒力因子主要指与致病过程相关的功能基因，例如毒素、黏附、侵袭、免疫逃逸、铁摄取系统、分泌系统及相关效应蛋白等。

**数据来源与处理**

*   本研究基于 VFDB 与非毒力基因对照集整理的基因层面二分类数据集。
    
*   正样本来自 **VFDB core dataset**中经实验验证或人工整理的代表性毒力相关基因，保留长度不超过 8 kb 的序列，最终得到 **4,570 条正样本**\[6\] 。
    
*   负样本来自 DEG essential genes 构建的非毒力对照集\[1\] ；
    
*   在构建负样本过程中，基于功能注释和描述信息剔除了潜在毒力相关条目，包括经典毒素/毒力因子、分泌系统/效应蛋白、荚膜、多糖及外层结构相关基因，随后按物种比例进行分层随机抽样，最终获得 **4,989 条非毒力必需基因**作为阴性对照。
    

**评测流程**

构建完成的数据集按照标签分层进行训练/测试划分，最终划分为 **8:2 train/test**。模型表征和下游头采用通用监督评测设置。

**评估指标**   AUROC、ACC 和 F1。

### 局部序列分类任务结果：

Genos-m 在 5 个局部序列分类任务中均位列前二，其中在抗生素抗性基因识别任务中，AUROC、ACC 和 F1 均取得最高结果（0.9896、0.9532 和 0.9531）。整体结果表明，在冻结 backbone 并仅训练轻量下游头的设定下，Genos-m 的局部序列表征能够稳定支持多类功能判别任务。MLP 分类任务的 AUC、ACC 和 F1 完整汇总见 Table S1。

**表 2. 局部序列分类任务预测性能比较（AUROC）**

| **任务** | **DNABERT-2** | **Nucleotide Transformer v2** | **Nucleotide Transformer v3** | **GenerAnno** | **Evo1-7B** | **Evo2-40B** | **Genos-m** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 启动子识别 | 0.6820 | 0.7773 | 0.7989 | 0.7804 | 0.7185 | **0.8504** | <ins>0.8460</ins> |
| 抗生素抗性基因识别 | 0.9176 | 0.9727 | 0.9529 | 0.9855 | 0.9313 | <ins>0.9884</ins> | **0.9896** |
| 毒力因子识别 | 0.7840 | 0.8942 | 0.7939 | 0.9301 | 0.8161 | **0.9617** | <ins>0.9548</ins> |
| 必需基因识别 | 0.6658 | 0.6921 | 0.7489 | 0.7953 | 0.7253 | **0.8619** | <ins>0.8534</ins> |
| 细菌基因六分类 | 0.9637 | 0.9866 | 0.9848 | 0.9884 | 0.9711 | **0.9937** | <ins>0.9932</ins> |

---

### 1.6 基因适应性预测（Gene Fitness Prediction，8 个细分任务）

**任务描述**   本任务根据基因序列预测其在特定营养或化学胁迫条件下的 gene fitness score，用于评估模型从基因序列中表征环境条件相关适应性信号的能力。原始 Prokaryotic Gener Tasks 将该任务定义为预测基因在特定实验条件下对微生物生存和适应性的贡献，因此该任务关注的是条件依赖的基因重要性，而不是泛化到所有环境的功能预测。

**数据来源与处理**

*   本任务基于 Fitness Browser 中的条件特异性基因适应性数据\[2\] ，并沿用 Generanno 的 gene fitness prediction 评测设定。当前 benchmark 选取 8 个代表性子任务，覆盖 nutrient utilization 与 chemical stress 两类适应性场景，包括：**Minimal media (glucose)、L-Arabinose (C)、Pyruvate (C)、D-Alanine (N)、Ammonium chloride (N)、L-Histidine (nutrient)、Cisplatin (stress) 和 Perchlorate (stress)**。
    

**表 3. Gene fitness prediction 子任务的环境条件分组与生物学含义**

| **环境类型** | **具体场景举例** | **生物学意义** |
| --- | --- | --- |
| **营养源变化** | 葡萄糖（基础碳源）、L-组氨酸（营养源）、丙酮酸（替代碳源）、阿拉伯糖（特殊碳源）、D-丙氨酸、氯化铵（氮源） | 评估模型对“基因 - 营养代谢关联”的理解，检验其是否能够区分不同碳源/氮源条件下的重要代谢基因。 |
| **化学压力** | 顺铂（药物毒性）、高氯酸盐（环境污染物） | 验证模型对“毒性抗性 / 应激响应基因”的预测能力，考察其在化学胁迫条件下的稳健性。 |

**评测流程**

*   采用通用监督评测设置。loss 为 MSE。
    

**评估指标**   Pearson correlation。

### 结果

在 8 个基因适应性回归子任务中，Genos-m 在 5 个子任务上取得对比模型中的最佳表现，其余 3 个子任务仅次于 Evo2-40B。该结果表明，Genos-m 的基因级序列表征能够有效支持条件特异性 gene fitness score 预测，并在不同营养和化学胁迫条件下保持较稳定的表现。逐任务 Pearson correlation 结果见表 4。

**表 4. 不同环境条件下的基因适应性预测性能比较（Pearson correlation）**

| **任务** | **DNABERT-2 (117M)** | **Nucleotide Transformer v2 (500M)** | **Nucleotide Transformer v3 (650M)** | **GenerAnno (500M)** | **Evo1-7B** | **Evo2-40B** | **Genos-m** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Ammonium chloride N** | 0.1196 | 0.3231 | 0.2981 | 0.4671 | 0.2386 | **0.5157** | <ins>0.5154</ins> |
| **Cisplatin stress** | 0.1253 | 0.1708 | 0.1308 | 0.2048 | 0.1232 | **0.3381** | <ins>0.3293</ins> |
| **D-Alanine N** | 0.0919 | 0.2394 | 0.2354 | 0.4101 | 0.2358 | **0.4734** | <ins>0.4314</ins> |
| **L-Arabinose C** | 0.1869 | 0.3654 | 0.3147 | 0.4807 | 0.2071 | <ins>0.5407</ins> | **0.5914** |
| **L-Histidine nutrient** | 0.1384 | 0.5440 | 0.3095 | 0.6340 | 0.2500 | <ins>0.6685</ins> | **0.7479** |
| **Min-media glucose** | 0.1718 | 0.5422 | 0.2988 | 0.6605 | 0.2578 | <ins>0.6936</ins> | **0.7513** |
| **Pyruvate C** | 0.1220 | 0.2805 | 0.2652 | <ins>0.4803</ins> | 0.1662 | 0.4661 | **0.5495** |
| **Perchlorate stress** | 0.1263 | 0.1695 | 0.0502 | 0.1494 | 0.0530 | <ins>0.2727</ins> | **0.2981** |

---

### 1.7 RNA 适应性零样本评测（RNA Fitness Prediction, Zero-shot）

**任务描述**   本任务基于 RNAGym 的 `RNAfitness` 子集，评估 Genos-m 在不进行任务特异性微调和下游预测头训练的情况下，对 RNA 序列变异效应的零样本排序能力。给定 RNA mutant 序列，模型直接基于语言模型分数对不同突变体进行排序，并通过与 RNAGym 实验 `DMS_score` / fitness readout 的相关性评估预测效果。该任务作为外部 zero-shot benchmark，用于评估 Genos-m 模型分数在原核相关 RNA assays 上是否与实验变异效应读数一致。

**数据来源与处理**

*   本任务使用 RNAGym 的 RNAfitness 数据，并从中筛选出 **13 个原核相关 assays**\[4\], 包括: Andreasson\_2020\_ribozyme, BCHB\_CHLTE\_Tsuboyama\_2023\_2KRU, BLAT\_ECOLX\_Firnberg\_2014, BLAT\_ECOLX\_Jacquier\_2013, CCDB\_ECOLI\_Adkar\_2012, ESTA\_BACSU\_Nutschel\_2020, F7YBW8\_MESOW\_Ding\_2023, IF1\_ECOLI\_Kelsic\_2016, MLAC\_ECOLI\_MacRae\_2023, PSAE\_PICP2\_Tsuboyama\_2023\_1PSE, Peri\_2022\_ribozyme, Q837P4\_ENTFA\_Meier\_2023, RNC\_ECOLI\_Weeks\_2023.
    
*   筛选依据为 assay 的 `dms_id` 或 description 中包含原核相关关键词。当前子集共包含 **54,384 条突变体序列**。
    

**评测流程**

*   采用 RNAfitness 零样本评测设置，不进行任务特异性微调，也不训练下游预测头。
    
*   其他对比模型结果来自 RNAGym 发布的 full-benchmark 结果，并在相同的 13 个 assays 上重新计算 assay-level Spearman 均值\[4\]。
    

**评估指标**   Spearman correlation、AUROC 和 MCC

### 结果

在 13 个原核相关 RNAfitness assays 子集中，Genos-m 取得较高的平均 Spearman 相关性，在公开对比模型中仅次于 Evo2-7B。该结果表明，Genos-m 在不进行任务特异性训练的情况下，能够对部分原核相关 RNA 突变体的实验 DMS / fitness readout 产生有效排序信号。

表5. Genos-m 与对比模型在 RNAGym RNAfitness 原核相关数据集中的零样本评测性能比较

| **Model** | **Spearman** | **AUC** | **MCC** |
| --- | --- | --- | --- |
| **Evo 2(7B)** | **0.39** | **0.691** | **0.284** |
| **Genos-m** | <ins>0.313</ins> | <ins>0.653</ins> | <ins>0.224</ins> |
| **Evo 1.5** | 0.289 | 0.64 | 0.208 |
| **Evo 1** | 0.264 | 0.626 | 0.179 |
| **RNAErnie** | 0.244 | 0.617 | 0.174 |
| **Nucl. Transformer** | 0.178 | 0.579 | 0.105 |
| **RNA-FM** | 0.168 | 0.577 | 0.103 |
| **RiNALMo** | 0.141 | 0.561 | 0.085 |
| **GenSLM** | 0.069 | 0.53 | 0.041 |

---

### 2. 区域 / 系统级长上下文任务测评 (Regional / System-Level Long-Context)

区域 / 系统级长上下文评测任务聚焦于生物合成基因簇（biosynthetic gene clusters, BGCs）相关序列。与单基因任务不同，BGC 通常由多个相邻基因共同构成，需要模型在较长上下文中整合基因共定位、模块边界和组合式功能信息。

#### 2.1 BGC 二分类（BGC Binary Classification）

**任务描述**   给定一段长基因组区域序列，判断其是否属于生物合成基因簇区域（BGC vs. non-BGC）。该任务用于评估模型是否能从多基因区域上下文中识别 BGC 相关的组合式功能信号。

> **BGC（生物合成基因簇）**：在染色体或质粒上紧密连锁排列的一组基因，通常共同编码一套酶、调控因子与转运蛋白，完成天然产物（次级代谢产物）的生物合成、调控与输出。

> **研究意义**：BGC的天然产物是抗生素/抗癌药/免疫抑制剂的重要来源；某些 BGC 也与污染物降解等生物修复相关；抗菌肽等也可作为抗生素替代路线。

**数据来源与处理**

*   正样本：MIBiG v4.0 实验验证 BGC（过滤真菌/未知/长度异常后保留 2,107）\[7\] 。
    
*   负样本：使用 antiSMASH 识别并排除含 BGC 的区域，再从不含 BGC 的 contig 中截取非 BGC 片段；截取时匹配正样本的长度和 CDS 数量分布，以降低序列长度或基因密度差异带来的混杂。最终保留 **2,107 条 non-BGC 区域**，正负样本比例为 1:1。
    
*   当前数据集按照 train/val/test = 8:1:1 划分，并保持类别分布一致。
    

**评测流程**

*   采用通用监督评测设置。
    
*   根据不同模型的最大上下文长度进行输入处理；对于超出模型上下文窗口的长序列，可采用截断或滑窗聚合策略。
    

**评估指标**   AUROC、ACC 和 F1。

#### 2.2 BGC 多标签分类（BGC Multi-lables Classification）

**任务描述**   给定一个已知 BGC 区域序列，预测其可能对应的生物合成类型（biosynthetic classes）。由于同一个 BGC 可能同时具有多个类型特征，例如同时包含 PKS 和 NRPS 相关模块，因此该任务定义为多标签分类任务（multilabel classification），而不是互斥的六分类任务。模型需要对每个 BGC 分别判断其是否属于 PKS、NRPS、ribosomal/RiPP、saccharide、terpene 和 other 六类标签中的一个或多个。

**数据来源与处理**

当前 benchmark 基于 MIBiG v4.0 整理 BGC 区域序列及其 biosynthetic class annotations\[7\] ，并将标签归并为 6 类：PKS、NRPS、ribosomal/RiPP、saccharide、terpene 和 other。每条 BGC 序列可对应一个或多个类别标签，因此各标签阳性样本数之和可大于 BGC 样本总数。该任务用于评估模型是否能够从 BGC 区域序列中提取与不同生物合成类别相关的多基因组织特征。

**评测流程**

*   当前数据集按照 train/val/test = 8:1:1 划分，并尽量保持各标签在训练、验证和测试集中的分布一致。
    
*   采用通用监督评测设置。由于类别分布不平衡，分类器损失函数采用 focal loss。
    
*   模型对每个 BGC 输出 6 个标签的预测概率。每个标签在验证集上独立搜索最优阈值，并据此将概率转换为多标签预测。
    

**评估指标**   AUROC、ACC 和 F1。

### BGC 序列分类结果

在区域级 BGC 评测中，Genos-m 在两个任务上均表现稳定。对于 BGC / non-BGC 区域识别，Genos-m 取得 AUROC = 0.9907，接近 Evo2-40B（0.9911），并高于 Nucleotide Transformer v3（0.9365）和 GenerAnno（0.9136），表明其能够有效支持给定候选区域上的 BGC 存在性判别。对于 BGC 类型多标签标注，Genos-m 取得 AUROC = 0.9216，处于第一梯队，说明其表征能够捕获已知 BGC 区域中与不同 biosynthetic classes 相关的多基因组织特征。BGC 两项 MLP 任务的 AUC、ACC 和 F1 完整汇总见 Table S1。

**表6. BGC 区域序列判别与BGC类型分类任务性能比较（AUROC）**

| **任务** | **DNABERT-2** | **Nucleotide Transformer v2** | **Nucleotide Transformer v3** | **GenerAnno** | **Evo1-7B** | **Evo2-40B** | **Genos-m** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| BGC 二分类 | 0.8551 | 0.9049 | 0.9365 | 0.9136 | 0.7984 | **0.9911** | <ins>0.9907</ins> |
| BGC 多标签分类 | 0.7897 | 0.7845 | 0.8271 | 0.8207 | 0.8101 | <ins>0.8951</ins> | **0.9216** |

---

### 3. 全基因组嵌入表征测评 (Whole-Genome Understanding)

#### 基因组表型预测（Genomic Phenotype Prediction）

**任务描述**   该任务基于 whole-genome embedding 进行菌株表型预测，用于评估模型是否能从全基因组序列中提取与细菌离散表型相关的全局信息。

给定一个基因组的 embedding，模型为每一种表型训练独立分类器，预测该基因组是否具有对应表型。

**数据来源与处理**

*   当前 benchmark 使用的表型数据来自 GIDEON 数据库，包含 **1,175 个基因组**和 **5 种离散分类表型**\[1\]。
    
*   数据按 **属（genus）水平**进行 train/val/test 划分，比例为 **0.7/0.1/0.2**，并在多个随机种子下重复评估。
    
*   次数：seeds=\[1, 2, 3\]
    

**表7：全基因组表征评估中的菌株二分类任务标签与样本统计**

| **Phenotype** | **Group** | **Label** | **Sample** | **Description** | **Source** |
| --- | --- | --- | --- | --- | --- |
| **gideon\_Gram positive** | +/- | 阳性/阴性 | 550/602 | 革兰氏染色特性 | gideon |
| **gideon\_Anaerobe** | +/- | 厌氧/非厌氧 | 322/789 | 氧气耐受性 | gideon |
| **gideon\_Motile** | +/- | 具备运动性/不具备运动性 | 292/746 | 运动性 | gideon |
| **gideon\_Spore formation** | +/- | 产孢/不产孢 | 95/1074 | 芽孢形成能力 | gideon |
| **gideon\_Beta hemolysis** | +/- | β-溶血阳性 / β-溶血阴性 | 52/631 | β-溶血活性 | gideon |

**评测流程**

*   评测采用 **linear probing**：Genos-m 先将每个细菌基因组按 1M bp 切分为 chunks，逐 chunk 推理获得 chunk-level embeddings，并对所有 chunk embeddings 取平均，得到 genome-level embedding。随后，针对每一种表型分别训练一个线性分类器。
    
*   在 seeds = \[1, 2, 3\] 下评估均值和波动范围。
    

**评估指标**   AUROC。

### 结果概述

Genos-m 基于 genome-level embedding 的菌株表型预测表现接近 Bacformer 这类面向全基因组尺度建模的模型，提示其 1M bp chunk 聚合表征能够保留部分与菌株离散表型相关的全局序列信号。

**表8：Genos-m 与其他模型在细菌表型预测任务上的性能比较（AUROC, mean ± SD）**

| **Phenotype/Model** | **Mistral-DNA** | **DNABERT-2** | **Nucleotide Transformer** | **ProkBERT** | **ESM-2** | **ESM-C** | **ProtBERT** | **gLM2** | **Bacformer** | **Genos-m** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **gideon Gram positive** | 65.58 ± 6.77 | 86.88 ± 5.47 | 88.12 ± 8.21 | 97.07 ± 3.41 | <ins>98.48 ± 1.69</ins> | **99.73 ± 0.29** | 97.50 ± 2.39 | 96.06 ± 4.29 | 97.61 ± 3.16 | 98.41 ± 1.77 |
| **gideon Anaerobe** | 77.21 ± 2.99 | 88.84 ± 4.07 | 90.90 ± 1.02 | 92.50 ± 1.52 | 96.14 ± 1.34 | 96.57 ± 2.93 | 97.92 ± 1.07 | 91.08 ± 1.84 | **98.92 ± 0.37** | <ins>98.14 ± 0.57</ins> |
| **gideon Motile** | 66.98 ± 9.54 | 76.09 ± 2.49 | 81.63 ± 3.84 | 64.59 ± 5.22 | 82.07 ± 4.29 | 74.70 ± 8.69 | 79.50 ± 6.52 | 63.98 ± 5.82 | **87.01 ± 9.71** | <ins>83.22 ± 3.52</ins> |
| **gideon Spore formation** | 52.59 ± 21.62 | 72.71 ± 16.94 | 79.25 ± 27.78 | 83.66 ± 22.84 | 89.55 ± 8.27 | 89.91 ± 12.13 | <ins>90.68 ± 7.52</ins> | 74.53 ± 30.76 | **90.85 ± 10.38** | 90.24 ± 14.58 |
| **gideon Beta hemolysis** | **63.40 ± 15.20** | <ins>62.26 ± 16.78</ins> | 60.54 ± 16.54 | 53.06 ± 5.76 | 54.80 ± 10.50 | 54.08 ± 6.86 | 53.71 ± 12.93 | 50.10 ± 5.57 | 52.05 ± 8.24 | 55.50 ± 1.59 |

## 结果汇总与简要结论

除 Fitness 外，各分类任务均报告 AUC；Fitness 报告 8 个基因适应性回归任务的平均 Pearson 相关系数。分类任务的 MLP AUC、ACC 和 F1 汇总见 Table S1；fitness 逐任务 Pearson 结果见表 4。

### 结论摘要

**Genos-m Benchmarks 旨在评估同一套预训练表征在不同微生物基因组尺度任务中的下游可用性。**结果显示，在冻结 backbone、仅训练轻量下游模型的统一设定下，Genos-m 表征能够支持短序列与基因级功能判别、条件特异性 gene fitness 回归、BGC 区域判别与类型分类，以及全基因组菌株表型预测。在 RNAfitness zero-shot 评测中，模型在无任务特异性训练的条件下，仍能对部分原核相关 RNA 突变体实验读数进行有效排序，提示其具备一定跨序列类型迁移能力。**上述结果支持 Genos-m 作为微生物基因组表征模型的有效性**，但结论应限定于当前 benchmark 的任务定义、数据划分和评测流程，不应外推为通用功能注释系统、通用 RNA 基础模型或开放环境下的通用菌株表型预测模型。

**在局部序列任务中**，Genos-m 在启动子识别、必需基因识别、抗生素抗性基因识别、毒力因子识别及基因类型分类等任务中均进入前三，表明其局部序列表征能够支持调控元件、功能区域及抗性/毒力相关序列信号的判别。在 8 个 gene fitness 回归子任务中，Genos-m 有 5 项达到当前最优表现，其余 3 项仅次于 Evo2-40B，进一步说明其基因级表征可为不同营养条件和化学胁迫下的连续型功能读数预测提供有效特征。

**在区域级长上下文任务中**，Genos-m 在 BGC / non-BGC 判别任务中接近当前最优表现，并在 BGC 多标签类型标注任务中达到当前最优表现，表明其能够从给定区域序列中整合多基因上下文信息，表征与 BGC 功能组织相关的序列特征。该结论限于给定区域序列的判别与分类任务，不等同于完整基因组中的 BGC 边界定位能力。

**在外部 RNAfitness zero-shot 评测中**，Genos-m 在 13 个原核相关 assays 上取得平均 Spearman = 0.313，在公开对比模型中排名第 2，仅次于 Evo2-7B，提示其模型分数能够在部分原核相关 RNA 突变体效应评估中提供有效排序信号。

**在全基因组 embedding 任务中**，Genos-m 将细菌基因组切分为 1M bp chunks，先生成 chunk-level embeddings，再通过均值聚合得到 genome-level embedding。在按属隔离的 GIDEON/BacBench 表型预测中，Genos-m 整体接近 Bacformer 这类面向全基因组尺度建模的模型，并在多个表型标签上达到第一梯队。该结果表明，其核酸级 chunk 聚合表征保留了与菌株基础表型相关的全局序列信号，但不应外推为开放环境下的通用菌株表型预测能力。

**综上所述**，Genos-m 的主要优势在于同一预训练表征在不同序列尺度和任务设定下的稳定表现。在冻结 backbone、仅训练轻量下游模型的监督评测中，Genos-m 表征可支持基因片段、BGC 区域和全基因组 embedding 等任务；在 RNAfitness zero-shot 评测中也能对部分原核相关 RNA 突变体实验读数提供排序信号。结合其面向人源相关微生物基因组的语料设计、百万碱基级上下文建模能力和较小的单次激活参数规模，Genos-m 可作为微生物基因组任务的基础表征模型，支持后续模型复用、任务扩展、标准化评测与下游应用探索。

---

**表 9 评测模型类型、参数规模与输入配置对比**

| **Model** | **Variant / Checkpoint** | **Objective** | **Params** | **dim** | **Max context** | **Input Type** |
| --- | --- | --- | --- | --- | --- | --- |
| **DNA Models** |  |  |  |  |  |  |
| **Mistral-DNA** | Mistral-DNA-v1-138M-bacteria | Autoregressive | 138 M | 768 | 512 | DNA |
| **DNABERT-2** \[8\] | DNABERT-2-117M | Masked | 117 M | 768 | 512 | DNA |
| **Nucleotide Transformer v2** \[9\] | nt-v2-250m-multi-species | Masked | 250 M | 768 | 2,048 | DNA |
| **Nucleotide Transformer v3** \[10\] | InstaDeepAI/NTv3\_650M\_pre | Masked | 500 M | 1536 | 1 M | DNA |
| **GenSLM\[11\]** | genslm\_2.5B\_patric | Autoregressive | 2.5B | 2560 | 2048 | DNA |
| **GenerAnno\[2\]** | GENERanno-eukaryote-0.5b-base | Masked | 500 M | 1024 | 8,192 | DNA |
| **Evo1-7B \[12\]** | evo-1-8k-base (1.1\_fix) | Autoregressive | 6.5 B | 4,096 | 8,192 | DNA |
| **Evo1.5-7B \[12\]** | evo-design/evo-1.5-8k-base | Autoregressive | 7 B | 4,096 | 8,192 | DNA |
| **Evo2-40B \[13\]** | evo2\_40b\_base | Autoregressive | 40 B | 4,096 | 8,192 | DNA |
| **ProkBERT \[14\]** | [neuralbioinfo/prokbert-mini-long](https://huggingface.co/neuralbioinfo/prokbert-mini-long) | Masked | 27 M | 384 | 4096 | DNA |
| **Genos-m** | Genos-m(cpt32k) | Autoregressive | 330 M(4.7B) | 1,024 | 1 M | DNA |
| **Protein Models** |  |  |  |  |  |  |
| **ESM-2 \[15\]** | esm2\_t12\_35M\_UR50D | Masked | 35 M | 480 | 1,024 | Single Protein |
| **ESM-C \[16\]** | esmc\_300m | Masked | 300 M | 960 | 1,024 | Single Protein |
| **ProtBERT  \[17\]** | Rostlab/prot\_bert | Masked | 420 M | 1,024 | 2048 | Single Protein |
| **Bacformer \[5\]** | bacformer-masked-complete-genomes | Masked | 27 M | 480 | 6,000 | Multiple Protein |
| **RNA Models** |  |  |  |  |  |  |
| **RNAErnie \[18\]** | RNAErnie | Masked | 86M | 768 | 512 | RNA |
| **RiNALMo \[19\]** | giga-v1 | Masked | 651M | 1280 | 1024 | RNA |
| **RNA-FM \[20\]** | rna\_fm\_t12 | Masked | 100M | 640 | 1024 | RNA |

**Table S1. MLP 分类任务预测性能比较（AUC、ACC 和 F1）**

| **任务** | **Metric** | **DNABERT-2 (117M)** | **Nucleotide Transformer v2 (500M)** | **Nucleotide Transformer v3 (650M)** | **GenerAnno (500M)** | **Evo1-7B** | **Evo2-40B** | **Genos-m** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 必需基因识别 | AUC | 0.6658 | 0.6921 | 0.7489 | 0.7953 | 0.7253 | **0.8619** | <ins>0.8534</ins> |
|  | ACC | 0.8704 | 0.8689 | 0.8726 | 0.8863 | 0.8678 | <ins>0.8919</ins> | **0.8981** |
|  | F1 | 0.5644 | 0.5241 | 0.5452 | 0.6733 | 0.5103 | <ins>0.7383</ins> | **0.7561** |
| 启动子识别 | AUC | 0.6820 | 0.7773 | 0.7989 | 0.7804 | 0.7185 | **0.8504** | <ins>0.8460</ins> |
|  | ACC | 0.6937 | 0.7471 | 0.7553 | 0.7502 | 0.7135 | **0.8011** | <ins>0.7970</ins> |
|  | F1 | 0.6021 | 0.6903 | 0.6990 | 0.6959 | 0.6210 | **0.7679** | <ins>0.7562</ins> |
| 细菌基因六分类 | AUC | 0.9637 | 0.9866 | 0.9848 | 0.9884 | 0.9711 | **0.9937** | <ins>0.9932</ins> |
|  | ACC | 0.7853 | 0.8887 | 0.8817 | 0.9053 | 0.8026 | **0.9322** | <ins>0.9280</ins> |
|  | F1 | 0.7821 | 0.8884 | 0.8814 | 0.9052 | 0.7980 | **0.9322** | <ins>0.9281</ins> |
| 抗生素抗性基因识别 | AUC | 0.9176 | 0.9727 | 0.9529 | 0.9855 | 0.9313 | <ins>0.9884</ins> | **0.9896** |
|  | ACC | 0.8274 | 0.9103 | 0.8818 | 0.9440 | 0.8557 | <ins>0.9459</ins> | **0.9532** |
|  | F1 | 0.8272 | 0.9102 | 0.8817 | 0.9440 | 0.8557 | <ins>0.9459</ins> | **0.9531** |
| 毒力因子识别 | AUC | 0.7840 | 0.8942 | 0.7939 | 0.9301 | 0.8161 | **0.9617** | <ins>0.9548</ins> |
|  | ACC | 0.7123 | 0.8086 | 0.6878 | 0.8572 | 0.7385 | **0.8980** | <ins>0.8792</ins> |
|  | F1 | 0.7116 | 0.8079 | 0.6838 | 0.8568 | 0.7381 | **0.8977** | <ins>0.8787</ins> |
| BGC 二分类 | AUC | 0.8551 | 0.9049 | 0.9365 | 0.9136 | 0.7984 | **0.9911** | <ins>0.9907</ins> |
|  | ACC | 0.7464 | 0.8081 | 0.8531 | 0.8389 | 0.7180 | <ins>0.9076</ins> | **0.9289** |
|  | F1 | 0.7358 | 0.8039 | 0.8510 | 0.8378 | 0.6893 | <ins>0.9069</ins> | **0.9287** |
| BGC 多标签分类 | AUC | 0.7897 | 0.7845 | 0.8271 | 0.8207 | 0.8101 | <ins>0.8951</ins> | **0.9216** |
|  | ACC | 0.2679 | 0.2919 | 0.3206 | 0.3732 | 0.3062 | <ins>0.5550</ins> | **0.5742** |
|  | F1 | 0.5286 | 0.5242 | 0.5581 | 0.5862 | 0.5257 | <ins>0.6512</ins> | **0.7384** |

**参考文献**

\[1\] Wiatrak M, Weimann A, Viñas Torné R, et al. **BacBench: Evaluating Genomic Language Models for Bacteria**\[EB/OL\]. OpenReview, ICLR 2026 Conference Withdrawn Submission, 2025\[2026-03-18\].

\[2\] Li Q, Wu W, Zhu Y, et al. **GENERanno: A Genomic Foundation Model for Metagenomic Annotation**\[J/OL\]. _bioRxiv_, 2025. DOI: 10.1101/2025.06.04.656517.

\[3\] Su W, Yang Y, Zhao Y, et al. **iPro-MP: a BERT-based model to predict multiple prokaryotic promoters**\[J\]. _Genome Biology_, 2025, 26(1): 353. DOI: 10.1186/s13059-025-03819-9.

\[4\] Arora R, Angelo M, Choe C A, et al. **RNAGym: Large-scale Benchmarks for RNA Fitness and Structure Prediction**\[J/OL\]. _bioRxiv_, 2025. DOI: 10.1101/2025.06.16.660049.

\[5\] Wiatrak M, Viñas Torné R, Ntemourtsidou M, et al. **A contextualised protein language model reveals the functional syntax of bacterial evolution**\[J/OL\]. _bioRxiv_, 2025. DOI: 10.1101/2025.07.20.665723.

\[6\] Zhou S, Liu B, Zheng D, et al. **VFDB 2025: an integrated resource for exploring anti-virulence compounds**\[J\]. Nucleic acids research, 2025, 53(D1): D871-D877.

\[7\] Zdouc M M, Blin K, Louwen N L L, et al. **MIBiG 4.0: advancing biosynthetic gene cluster curation through global collaboration**\[J\]. Nucleic acids research, 2025, 53(D1): D678-D690.

\[8\]Zhou Z, Ji Y, Li W, et al. **Dnabert-2: Efficient foundation model and benchmark for multi-species genome. arXiv\[J\].** arXiv preprint arXiv:2306.15006, 2023

\[9\]Dalla-Torre H, Gonzalez L, Mendoza-Revilla J, et al. **Nucleotide Transformer: building and evaluating robust foundation models for human genomics**. _Nature Methods_, 2025, 22(2):287–297. DOI: 10.1038/s41592-024-02523-z.

\[10\]Boshar S, Evans B, Tang Z, et al. **A foundational model for joint sequence-function multi-species modeling at scale for long-range genomic prediction**. _bioRxiv_, 2025. DOI: 10.64898/2025.12.22.695963.

\[11\]Zvyagin M, Brace A, Hippe K, et al. **GenSLMs: Genome-scale language models reveal SARS-CoV-2 evolutionary dynamics**. _The International Journal of High Performance Computing Applications_, 2023, 37(6):683–705. DOI: 10.1177/10943420231201154.

\[12\]Nguyen E, Poli M, Durrant M G, et al. **Sequence modeling and design from molecular to genome scale with Evo**. _Science_, 2024. DOI: 10.1126/science.ado9336.

\[13\]Brixi G, Durrant M G, Ku J, et al. **Genome modelling and design across all domains of life with Evo 2**. _Nature_, 2026.

\[14\]Ligeti B, Szepesi-Nagy I, Bodnár B, et al. **ProkBERT family: genomic language models for microbiome applications**. _Frontiers in Microbiology_, 2024, 14:1331233. DOI: 10.3389/fmicb.2023.1331233.

\[15\]Lin Z, Akin H, Rao R, et al. **Evolutionary-scale prediction of atomic-level protein structure with a language model**. _Science_, 2023, 379(6637):1123–1130. DOI: 10.1126/science.ade2574.

\[16\]**ESM Cambrian: Revealing the mysteries of proteins with unsupervised learning**. EvolutionaryScale website, 2024；

\[17\]Elnaggar A, Heinzinger M, Dallago C, et al. **ProtTrans: Towards Cracking the Language of Life’s Code Through Self-Supervised Deep Learning and High Performance Computing**. _IEEE Transactions on Pattern Analysis and Machine Intelligence_, 2021/2022. DOI: 10.1109/TPAMI.2021.3095381.

\[18\]Wang N, et al. **Multi-purpose RNA language modelling with motif-aware pretraining and type-guided fine-tuning**. _Nature Machine Intelligence_, 2024. DOI: 10.1038/s42256-024-00836-4.

\[19\]Penić R J, Vlašić T, Huber R G, et al. **RiNALMo: general-purpose RNA language models can generalize well on structure prediction tasks**. _Nature Communications_, 2025, 16:5671. DOI: 10.1038/s41467-025-60872-5.

\[20\]Chen J, Hu Z, Sun S, et al. **Interpretable RNA Foundation Model from Unannotated Data for Highly Accurate RNA Structure and Function Predictions**. arXiv / bioRxiv, 2022.
