# Genos-m Training Scripts

This directory contains the Megatron-LM pre-training launcher for Genos-m-4.7B.

Genos-m-4.7B is a sparse Mixture-of-Experts Transformer for human-associated microbial genome modeling. It has 4.7B total parameters, activates about 0.33B parameters per forward pass, and supports context lengths up to 1M nucleotide tokens. The model uses a single-nucleotide DNA tokenizer, grouped-query attention, RoPE with a large rotary base, SwiGLU, RMSNorm, FlashAttention, and Top-2 MoE routing over 32 experts.

## Model Configuration

| Component | Genos-m-4.7B |
|-----------|--------------|
| Layers | 12 |
| Hidden size | 1024 |
| FFN hidden size | 4096 |
| Attention heads | 16 |
| Query groups (GQA) | 8 |
| MoE experts | 32 |
| Router | Top-2 |
| Max context | 1M tokens |
| Active parameters | 0.33B |
| Total parameters | 4.7B |

## Training

### Objective

Genos-m is trained with next-token prediction on DNA sequences. Training uses progressive context scaling, with the provided launcher configured for an 8K sequence length and a 1M maximum position range.

### Data

The pre-training corpus contains about 1.2T nucleotide tokens after quality control and stratified dynamic sampling. It combines GTDB R220 representative prokaryotic genomes, public human-associated microbial genomes, in-house high-quality human gut MAGs, and UHGV human gut phage genomes.

The launcher expects `DATA_PATH` to point to Megatron-LM indexed dataset prefix(es), and `TOKENIZER_MODEL` to point to the character-level DNA tokenizer.

### Infrastructure
- **Framework**: Megatron-LM on 128 GPUs
- **Parallelism**: 5D strategy (TP, CP, DP, PP, EP)
- **Batch**: Global 1024, Micro 4
- **Optimizer**: AdamW (distributed sharded)
- **Learning Rate**: up to 1e-4, cosine decay, 5-10% warmup
- **Precision**: BF16 for most compute, FP32 for softmax/gradients/routing

### Key Optimizations
- **MoE Load Balancing**: Aux loss (1e-3) + Z-loss (1e-3)
- **Communication**: Grouped GEMM, AllToAll dispatch, overlapped gradient reduction
- **Memory**: Flash Attention, sequence parallelism, distributed optimizer

## Launcher

Use `Genos-m-4.7B-pretrain.sh` from the Megatron-LM root directory, where `pretrain_gpt.py` is available:

```bash
bash Scripts/Genos-m-4.7B-pretrain.sh \
  <wandb_api_key> \
  <wandb_run_name> \
  <train_samples> \
  <checkpoint_path> \
  <tokenizer_model> \
  <data_path>
```

Built with [Megatron-LM](https://github.com/NVIDIA/Megatron-LM).
