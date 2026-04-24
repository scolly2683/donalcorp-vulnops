# OpenMythos

**Source**: https://github.com/kyegomez/OpenMythos  
**Ingested**: 2026-04-19  
**License**: MIT  
**Language**: Python  
**Stars**: 88 | **Forks**: 17 | **Contributors**: 1 (kyegomez)

---

## Summary

Community-driven, open-source theoretical reconstruction of the **Claude Mythos** architecture. Not affiliated with Anthropic. Based solely on publicly available research and speculation.

Core thesis: Mythos achieves superior reasoning via a **Recurrent-Depth Transformer (RDT)** — recycling a subset of transformer layers through multiple forward-pass iterations rather than stacking hundreds of unique layers.

> "Same weights. More loops. Deeper thinking."

At 770M parameters, the looped model reportedly achieves quality comparable to a 1.3B fixed-depth transformer (~50% parameter efficiency gain).

---

## Architecture

Three-stage pipeline:

```
Input → [Prelude] → [Recurrent Block × T loops] → [Coda] → Output
```

| Stage | Description |
|-------|-------------|
| **Prelude** | Standard transformer blocks, run once |
| **Recurrent Block** | Single transformer block looped up to `max_loop_iters` times with shared weights |
| **Coda** | Standard transformer blocks for final refinement, run once |

### Attention Mechanisms

- **GQA** (Grouped Query Attention): fewer KV heads than Q heads
- **MLA** (Multi-Latent Attention): low-rank KV compression → ~10–20× memory reduction

### Mixture of Experts (MoE)

- Fine-grained routed experts selected via top-K routing
- Shared experts always activated for every token
- Load-balancing bias for even compute distribution

### Recurrent Stability

- **LTI injection**: Spectral radius < 1 enforced by construction via discretized diagonal state matrices — prevents exploding hidden states over loops
- **ACT halting**: Adaptive Computation Time allows early exit per position when convergence is detected
- **LoRA adapters**: Depth-wise parameter adjustment per loop iteration

### Positional Encoding

- **RoPE** (Rotary Position Embeddings) on attention heads
- **Loop-index embeddings**: Sinusoidal signals distinguish recurrence iterations, enabling depth extrapolation (train on N loops, infer at N+k loops)

### Other Details

- **RMSNorm** throughout
- **KV caching** for autoregressive generation
- **Weight tying** between embedding and output projection
- **Causal masking** for autoregressive prediction

---

## Key Hypotheses About Mythos

1. **Implicit multi-hop reasoning** via iterative latent updates (no explicit chain-of-thought output)
2. **Depth-based inference scaling**: more loops at test time = more computation = better answers
3. LTI stability constraints solve the training instability that previously made deep recurrence impractical

---

## Repository Structure

```
OpenMythos/
├── open_mythos/
│   ├── __init__.py
│   └── main.py          ← Full RDT implementation (GQA/MLA, MoE, ACT, LoRA, RoPE)
├── docs/                ← API documentation
├── tests/
├── example.py           ← Usage example (see below)
├── test_main.py
├── requirements.txt
├── README.md
└── LICENSE
```

---

## Installation

```bash
pip install git+https://github.com/kyegomez/OpenMythos.git
# or clone and:
pip install torch>=2.1.0
```

---

## Usage Example

```python
import torch
from open_mythos.main import OpenMythos, MythosConfig

attn_type = "mla"  # or "gqa"

base = {
    "vocab_size": 1000,
    "dim": 256,
    "n_heads": 8,
    "max_seq_len": 128,
    "max_loop_iters": 4,
    "prelude_layers": 1,
    "coda_layers": 1,
    "n_experts": 8,
    "n_shared_experts": 1,
    "n_experts_per_tok": 2,
    "expert_dim": 64,
    "lora_rank": 8,
    "attn_type": attn_type,
}

# MLA config (more complex KV compression)
cfg = MythosConfig(
    **base,
    n_kv_heads=8,
    kv_lora_rank=32,
    q_lora_rank=64,
    qk_rope_head_dim=16,
    qk_nope_head_dim=16,
    v_head_dim=16,
)

model = OpenMythos(cfg)

# Forward pass with 4 recurrent loops
ids = torch.randint(0, cfg.vocab_size, (2, 16))
logits = model(ids, n_loops=4)

# Generation with 8 loops (depth extrapolation)
out = model.generate(ids, max_new_tokens=8, n_loops=8)

# Verify stability: spectral radius must be < 1
A = model.recurrent.injection.get_A()
print(f"Spectral radius ρ(A) max: {A.max().item():.4f}")
```

---

## Dependencies

```
torch>=2.1.0
pytest>=7.0.0
```

---

## Related Concepts

- [[Recurrent Depth Transformers]]
- [[Mixture of Experts]]
- [[Adaptive Computation Time]]
- [[Multi-Latent Attention]]
- [[Rotary Position Embeddings]]
- [[LoRA]]

---

## Tags

#ai/architecture #ai/transformers #ai/reasoning #open-source #python #research/speculative
