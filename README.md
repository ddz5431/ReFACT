# ReFACT: A Benchmark for Scientific Confabulation Detection with Positional Error Annotations
[![arXiv](https://img.shields.io/badge/arXiv-2509.25868-b31b1b.svg)](https://arxiv.org/abs/2509.25868)
![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Dataset](https://img.shields.io/badge/Dataset-1001%20samples-green.svg)

**ReFACT** (Reddit False And Correct Texts) is a benchmark dataset for evaluating how Large Language Models detect, localize, and correct scientific confabulation.

## Overview

Scientific confabulation represents a critical challenge in LLM deployment - the generation of fluent, plausible, and contextually appropriate text that is nonetheless factually incorrect. Unlike obvious factual errors, scientific confabulations are subtle, domain-specific, and often require expert knowledge to detect.

The dataset contains 1,001 expert-annotated question-answer pairs spanning diverse scientific domains, enabling fine-grained evaluation of LLMs' factuality capabilities. 

**Version**: v1.0 (September 2025)

### Key Features

- **1,001 expert-annotated samples** across 10+ scientific domains
- **Three-tier evaluation framework**: Detection → Localization → Correction  
- **Two transformation types**: Logical Negation and Entity Replacement
- **Human-verified quality** with multi-annotator consensus
- **Real-world grounding** from r/AskScience community content

## Benchmark Results

Even state-of-the-art models struggle significantly with scientific confabulation:

| Model | Ind Judgment | Comp Judgment | Neg Local | Ent Local | Ent Correct | Avg Acc |
|-------|--------------|---------------|-----------|-----------|-------------|---------|
| GPT-4o | 0.67/0.67 | 0.60/0.53 | 0.66/0.57 | 0.47/0.38 | 0.28 | 0.54 |
| Gemma-3-27B | 0.71/0.72 | 0.56/0.54 | 0.61/0.46 | 0.46/0.29 | 0.24 | 0.52 |
| Llama-3.3-70B | 0.67/0.73 | 0.50/0.39 | 0.69/0.61 | 0.13/0.24 | 0.16 | 0.43 |

*Ind Judgment: Independent Detection (Accuracy/F1), Comp Judgment: Comparative Detection (Accuracy/F1), Neg Local: Negation Localization (Accuracy/IoU), Ent Local: Entity Localization (Accuracy/IoU), Ent Correct: Entity Correction (Exact Match), Avg Acc: Average Accuracy across tasks.

**Key Findings:**
- **Detection shows limitations**: Independent judgment achieves 49-71% accuracy, but comparative judgment (directly choosing between factual vs. confabulated answers) drops to near-random performance (48-60%), even for the best-performing models.
- **Entity localization is substantially harder**: Accuracy ranges from 4-47% for entity localization compared to 0-69% for negation localization across all models. Even the best-performing model (GPT-4o) achieves only 47% on entity localization versus 66% on negation detection, revealing particular difficulty with subtle terminological substitutions.
- **Correction remains largely unsolved**: Exact match accuracy ranges from 0-28%, showing models struggle to generate accurate factual corrections even when error locations are explicitly marked.
- **Implications for LLM-as-Judge**: Near-random comparative judgment performance (48-60%) raises fundamental concerns about the reliability of LLM-as-Judge paradigms for evaluation tasks requiring nuanced discrimination.

## Installation & Requirements

### Prerequisites
```bash
# Python 3.9+ required
pip install pandas
```
### Download

The dataset is available in this repository in two versions:

- **`dataset_multi_error.jsonl`**: Multi-error version with 1,001 samples - 165 samples contain multiple error spans *(primary dataset for official benchmarking)*
- **`dataset_single_error.jsonl`**: Single-error version with 1,251 samples - each sample contains exactly one error span *(simplified variant for development)*

Both versions are also available via Hugging Face Hub (coming soon).

## Dataset Structure

Both dataset versions follow the same format but differ in error complexity:

### Sample Format (JSONL)

```json
{
  "sample_id": "0713184c...",
  "question": "Why are there no saltwater amphibians?",
  "correct_answer": "There is one saltwater amphibian: the crab-eating frog...allowing it to retain osmotic equilibrium with a saltwater environment.",
  "error_type": "swap",
  "confabulated_answer": "There is one saltwater amphibian: the crab-eating frog...allowing it to retain osmotic disequilibrium with a saltwater environment.",
  "error_spans": "...allowing it to retain <swap>osmotic disequilibrium</swap> with a saltwater environment..."
}
```

### Key Fields

- **`question`**: Original scientific question from r/AskScience
- **`correct_answer`**: Factually correct answer
- **`confabulated_answer`**: Answer with introduced confabulation
- **`error_spans`**: Confabulated answer annotated with error spans using `<neg>...</neg>` or `<swap>...</swap>` tags
- **`error_type`**: Transformation type (`neg` for negation, `swap` for entity replacement)

### Transformation Types

The dataset contains two types of scientific confabulations:

- **Negation (`neg`)**: Logical negation of factual statements while preserving plausibility
  - *Example*: "As we get older we **lose** the ability..." → "As we get older we **gain** the ability..."

- **Entity Replacement (`swap`)**: Substitution of technical terms with plausible but incorrect alternatives  
  - *Example*: "...supported by the **Sodium/Potassium ATPase**..." → "...supported by the **Calcium/Magnesium Pump**..."

### Dataset Versions

ReFACT provides two complementary versions derived from the same 1,001 expert-annotated samples:

**Multi-Error Version** (`dataset_multi_error.jsonl`) - *Primary dataset as described in the paper*:
- 1,001 samples total  
- 165 samples (16.5%) contain multiple error spans
- Multiple errors occur when an entity and its coreferences are replaced throughout the text
- Tests whether models can consistently identify all instances of the same incorrect entity
- More challenging and realistic evaluation of model robustness
- **Recommended for official benchmarking and paper comparisons**

**Single-Error Version** (`dataset_single_error.jsonl`) - *Simplified evaluation variant*:
- 1,251 samples total
- Each sample contains exactly one error span
- Useful for initial method development and isolating detection/localization performance
- Simplifies evaluation by removing the coreference tracking requirement

## Quick Start

### Loading the Dataset

```python
import json
import pandas as pd

# Load multi-error version (primary dataset, recommended for benchmarking)
with open('refact_multi_error.jsonl', 'r') as f:
    multi_error_data = [json.loads(line) for line in f]

# Load single-error version (optional, for simplified evaluation)
with open('refact_single_error.jsonl', 'r') as f:
    single_error_data = [json.loads(line) for line in f]

# Convert to DataFrames
df_multi = pd.DataFrame(multi_error_data)
df_single = pd.DataFrame(single_error_data)

print(f"Multi-error dataset (primary): {len(df_multi)} samples")
print(f"Single-error dataset (simplified): {len(df_single)} samples")
print(f"Error types: {df_multi['error_type'].value_counts().to_dict()}")
```

### Data Splits

No pre-defined splits are provided. Researchers should create their own train/validation/test splits based on their experimental needs.

## Evaluation Framework

ReFACT enables evaluation across three progressive tasks:

### 1. Confabulation Detection
Binary classification of scientific content as factual or confabulated.

### 2. Error Localization  
Identifying specific spans within text that contain factual errors.

### 3. Content Correction
Generating corrections to restore factual accuracy.

### Evaluation Methodology
The benchmark results were obtained using **zero-shot prompting** with task-specific templates:

- **Detection Task**: Binary classification (Factual vs. Confabulated)
  - Metrics: Accuracy and F1
  
- **Localization Task**: Identify error spans in confabulated answers
  - Metrics: Accuracy and Intersection over Union (IoU) for span overlap
  
- **Correction Task**: Generate corrections for identified errors
  - Metric: Exact Match against ground truth corrections

The evaluation framework is designed to be:
- **Independent**: Each task evaluated separately to avoid error compounding
- **Comprehensive**: Covers detection, localization, and correction
- **Robust**: Uses strict metrics (IoU, Exact Match) to avoid misleading scores


## Dataset Construction

### Source Data
- **Origin**: r/AskScience subreddit (23M+ members)
- **Quality Control**: High-scoring questions and answers only.
- **Filtering**: 500-1000 character length, score ≥4.

### Transformation Pipeline
1. **Fact Extraction**: Identify factual statements using Gemma-2-27B-it.
2. **Transformation**: Apply negation or entity replacement using Gemma-2-27B-it.
3. **Selection**: Choose most convincing confabulated version.
4. **Human Annotation**: Multi-annotator verification (72.5% pairwise agreement rate).

### Quality Assurance
- 3 independent annotators per sample.
- Consensus requirement (≥2/3 agreement).
- Detailed annotation guidelines.
- 58% transformation success rate after iterative refinement.

## Repository Structure

```
ReFACT/
├── dataset_single_error.jsonl       # Single-error version (1,251 samples)
├── dataset_multi_error.jsonl        # Multi-error version (1,001 samples)
├── README.md                        # This file  
├── LICENSE                          # MIT license
├── CITATION.cff                     # Citation metadata (CFF format)
├── CITATION.bib                     # BibTeX citation
├── requirements.txt                 # Python dependencies
```

## FAQ

**Q: What makes ReFACT different from other factuality benchmarks?**
A: ReFACT focuses specifically on subtle scientific confabulations that require domain expertise to detect, with full human verification and span-level annotations.

**Q: How were the transformations created?**
A: We used a systematic pipeline with Gemma-2-27B-it for fact extraction and transformation generation, followed by rigorous human validation (3 annotators per sample, ≥2/3 consensus) to ensure realistic, challenging confabulations.

**Q: Which dataset version should I use?**
A: Use `dataset_multi_error.jsonl` for **official benchmarking and paper comparisons** (this is the primary dataset described in the paper with 1,001 samples). Use `dataset_single_error.jsonl` for initial evaluation, method development, and ablation studies where single-error simplification is beneficial.

**Q: Can I use this for commercial applications?**
A: Yes, the dataset is released under MIT license for both research and commercial use.

## License

This dataset is released under the [MIT License](LICENSE). You are free to use, modify, and distribute this dataset for any purpose, including commercial use.

## Citation

If you use ReFACT in your research, please cite our paper:

```bibtex
@article{wang2025refact,
  title={{ReFACT}: A Benchmark for Scientific Confabulation Detection with Positional Error Annotations},
  author={Wang, Yindong and Prei{\ss}, Martin and Bague{\~n}o, Margarita and Hoffbauer, Jan Vincent and Ghajar, Abdullatif and Buz, Tolga and de Melo, Gerard},
  journal={arXiv preprint arXiv:2509.25868},
  year={2025},
  url={https://arxiv.org/abs/2509.25868}
}
```

A standalone BibTeX file is available at [`CITATION.bib`](CITATION.bib).