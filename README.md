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
- **Comparative judgment collapses to near-random performance (48-60%)**: While models achieve moderate accuracy when judging answers independently (49-71%), they fail to reliably distinguish between factual and confabulated versions when directly compared. This challenges the reliability of LLM-as-Judge paradigms for tasks requiring nuanced factual discrimination.
- **Domain-specific substitutions are substantially harder to detect than logical negations**: Even the best-performing model (GPT-4o) achieves only 47% accuracy on entity localization versus 66% on negation localization. Models struggle to identify subtle terminological errors like "DNA"→"RNA" that require domain expertise in scientific contexts.
- **Correction remains largely unsolved (0-28% accuracy)**: Even when error locations are explicitly provided, models cannot reliably generate factual corrections, demonstrating that error detection does not imply understanding.

## Quick Start

```bash
# Python 3.9+ required
pip install pandas
```

**Dataset files:**
- `refact_multi_error.jsonl` (1,001 samples) - **Primary dataset for benchmarking**
- `refact_single_error.jsonl` (1,251 samples) - Simplified variant for development

### Dataset Format

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

**Fields:**
- `question` - Scientific question from r/AskScience
- `correct_answer` / `confabulated_answer` - Factual vs. confabulated versions
- `error_spans` - Annotated with `<neg>...</neg>` or `<swap>...</swap>` tags
- `error_type` - Either `neg` (negation) or `swap` (entity replacement)

**Transformation types:**
- **Negation**: "lose ability" → "gain ability"
- **Entity Swap**: "DNA" → "RNA"

**Dataset versions:**
- **Multi-error** (1,001 samples): 16.5% contain multiple error spans from coreference replacement. Use for official benchmarking.
- **Single-error** (1,251 samples): Each sample has exactly one error span. Use for development.

### Loading the Dataset

```python
import json
with open('refact_multi_error.jsonl', 'r') as f:
    data = [json.loads(line) for line in f]
```

## Evaluation Tasks

ReFACT evaluates three progressive capabilities:

1. **Detection** - Binary classification (factual vs. confabulated)  
   *Metrics: Accuracy, F1*

2. **Localization** - Identify error spans  
   *Metrics: Accuracy, IoU*

3. **Correction** - Generate factual corrections  
   *Metric: Exact Match*


## Dataset Construction

**Source:** r/AskScience subreddit (high-scoring Q&A pairs, 500-1000 chars)

**Pipeline:**
1. Fact extraction (Gemma-2-27B-it)
2. Transformation (negation or entity swap)
3. Human verification (3 annotators, ≥2/3 consensus, 72.5% agreement)

## Repository Structure

```
ReFACT/
├── refact_single_error.jsonl       # Single-error version (1,251 samples)
├── refact_multi_error.jsonl        # Multi-error version (1,001 samples)
├── human_annotation_guideline.md   # Annotation guidelines
├── README.md                       # This file  
├── LICENSE                         # MIT license
├── CITATION.bib                    # BibTeX citation
```

## FAQ

**Q: What makes ReFACT unique?**  
A: Unlike benchmarks that rely on synthetic corruption or automatic perturbations of Wikipedia/LLM-generated text, ReFACT combines real community Q&As from r/AskScience with systematic transformations and rigorous human verification. This produces subtle, plausible confabulations that require domain expertise to detect. Additionally, ReFACT provides span-level error annotations and enables three-tier evaluation (detection, localization, correction) - not just binary classification.

**Q: Which version should I use?**  
A: Use `refact_multi_error.jsonl` for official benchmarking; `refact_single_error.jsonl` for development.

## License & Citation

Released under [MIT License](LICENSE). If you use ReFACT, please cite:

```bibtex
@article{wang2025refact,
  title={{ReFACT}: A Benchmark for Scientific Confabulation Detection with Positional Error Annotations},
  author={Wang, Yindong and Prei{\ss}, Martin and Bague{\~n}o, Margarita and Hoffbauer, Jan Vincent and Ghajar, Abdullatif and Buz, Tolga and de Melo, Gerard},
  journal={arXiv preprint arXiv:2509.25868},
  year={2025}
}
```