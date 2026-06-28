# Hierarchical Explanation Graph (HEG)
### A Causal Framework for Explainability in Hierarchical LLM-Based Multi-Agent Systems

**Adam Khald** — Independent Researcher | ad.khald@edu.umi.ac.ma

**Published in Zenodo** — https://doi.org/10.5281/zenodo.21002532 

---

## Overview

LLM agents in hierarchical multi-agent pipelines can produce fluent, confident self-explanations that are **causally disconnected from their actual decision process** — a phenomenon analogous to confabulation in cognitive science.

This paper introduces the **Hierarchical Explanation Graph (HEG)**, a two-level explainability framework that separates:

- **Intentional explanations** — what each layer *reports* about its decision
- **Causal explanations** — what controlled perturbation experiments *empirically show* drove the decision

The gap between these two levels is what HEG is built to detect and quantify.

---

## Pipeline Architecture

The evaluated pipeline consists of 6 sequential layers:

| Layer | Role |
|-------|------|
| `l0` Global Planner | Strategic goal decomposition |
| `l1` Mid-Level Planner | Subtask breakdown and workflow ordering |
| `l2` Memory System | Retrieval of relevant memory chunks |
| `l3` Research Agent | Data collection and sourcing |
| `l4` Analysis Agent | Insight synthesis and conclusions |
| `l5` Critic Evaluator | Quality scoring and final answer |

---

## Method

### Dual-Orchestrator Architecture

- **Orchestrator A** runs the normal pipeline, logging all intermediate states and layer-level self-explanations simultaneously (not post-hoc).
- **Orchestrator B** receives those states, perturbs each layer's output one at a time via the Perturbation Engine, re-runs downstream layers, and records the resulting final output.

This yields **6 perturbation experiments per task** at linear complexity O(n), not exponential.

### Perturbation Engine

A dedicated LLM component that generates minimal, semantically meaningful, layer-aware interventions — e.g., removing a constraint at the Global Planner, flipping a conclusion at the Analysis Agent, or downgrading a source reliability at the Research Agent.

### Causal Weight Computation

For each perturbation experiment `i`, cosine divergence between the original and perturbed final outputs is computed:

```
D_i = 1 - cos(emb(A), emb(B_i))
```

Causal weights are then normalized so they sum to 1:

```
W_i = D_i / Σ D_j
```

A layer with `W_i > 1/6 ≈ 0.167` exerts above-uniform causal influence.

---

## Key Results

Evaluated across **7 tasks** (3 simple, 4 multi-step) for a total of **42 perturbation experiments**.

### Confabulation Rates by Layer

| Layer | Confabulation Rate |
|-------|--------------------|
| Critic Evaluator | **100%** |
| Analysis Agent | 43% |
| Global Planner | 29% |
| Memory System | 14% |
| Mid-Level Planner | **0%** |
| Research Agent | **0%** |

### Notable Findings

- **The Critic Evaluator Paradox**: the layer that produces and justifies the final answer has near-zero causal weight (`W5 ≈ 0.000`) in every task. Its scores change when perturbed, but the final answer does not — it was already determined by upstream layers.
- **Simple tasks are front-loaded**: the Global Planner dominates (up to `W0 = 0.570` for Email Drafting). Optimizing the Global Planner prompt yields disproportionate returns.
- **Multi-step tasks distribute causality**: all layers from Global Planner through Analysis Agent contribute meaningfully (`W_i ≈ 0.17–0.24`), validating the architectural justification for deep pipelines.
- **Execution layers self-report faithfully**: the Research Agent and Mid-Level Planner confabulate 0% of the time, consistent with their concrete, structured inputs.

---

## Implications for Agent Design

1. **Direct explanation requests to low-confabulation layers** (Research Agent, Mid-Level Planner) rather than the final output layer.
2. **Use periodic perturbation experiments as a runtime faithfulness check** — flagging when a layer's self-report diverges from its measured causal contribution.
3. **Task complexity determines causal topology**: don't assume the same layer drives output across task types.

---

## Limitations & Future Work

- All layers used the same backbone model (GPT-4o); cross-model confabulation patterns may differ
- Minimal perturbation is enforced via prompting, not a formal perturbation budget
- Cosine divergence may miss structural output changes that are semantically neutral
- Specialized domains (medical, legal, scientific) not yet evaluated

Planned extensions: cross-model evaluation, formal perturbation budgets, integration with token-level mechanistic interpretability, real-time faithfulness monitoring.

---

## Citation

```bibtex
@article{khald2025heg,
  title     = {Hierarchical Explanation Graph: A Causal Framework for Explainability
               in Hierarchical LLM-Based Multi-Agent Systems},
  author    = {Khald, Adam},
  year      = {2025},
  note      = {Independent Researcher, ENSAM Meknès},
  email     = {ad.khald@edu.umi.ac.ma}
}
```

---

## Dependencies

- `sentence-transformers` — `all-MiniLM-L6-v2` for semantic embeddings
- `openai` — GPT-4o as backbone LLM
- `networkx` — HEG graph construction and visualization
