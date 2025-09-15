# ReFACT: A Benchmark for Scientific Confabulation Detection with Positional Error Annotations

![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Dataset](https://img.shields.io/badge/Dataset-1001%20samples-green.svg)
![Paper](https://img.shields.io/badge/Paper-Preprint-red.svg)

**ReFACT** (Reddit False And Correct Texts) is a benchmark dataset for detecting, localizing, and correcting scientific confabulation in Large Language Models (LLMs).

## 🎯 Overview

Scientific confabulation represents a critical challenge in LLM deployment - the generation of fluent, plausible, and contextually appropriate scientific text that is nonetheless factually incorrect. Unlike obvious factual errors, scientific confabulations are subtle, domain-specific, and often require expert knowledge to detect.

The dataset contains 1,001 expert-annotated question-answer pairs spanning diverse scientific domains, designed to evaluate fine-grained factuality in scientific discourse.

**Version**: v1.0 (September 2025)

### Key Features

- 📊 **1,001 expert-annotated samples** across 10+ scientific domains
- 🔍 **Three-tier evaluation framework**: Detection → Localization → Correction  
- 🧬 **Two transformation types**: Logical Negation and Entity Replacement
- ✅ **Human-verified quality** with multi-annotator consensus
- 🌐 **Real-world grounding** from r/AskScience community content

## 📈 Benchmark Results

Even state-of-the-art models struggle significantly with scientific confabulation:

| Model | Average Accuracy | Detection | Localization | Correction |
|-------|------------------|-----------|--------------|------------|
| GPT-4o | 54% | 67% | 47% | 28% |
| Gemma-3-27B | 52% | 71% | 46% | 24% |
| Llama-3.3-70B | 43% | 67% | 13% | 16% |

## 📦 Installation & Requirements

### Prerequisites

```bash
# Python 3.9+ required
pip install pandas>=1.3.0
pip install datasets>=2.0.0  
```

### Download

The dataset is available in this repository in two versions:

- **`dataset_single_error.jsonl`** (~2.4MB): Single-error version with 1,251 samples - each sample contains exactly one error span *(recommended for initial evaluation)*
- **`dataset_multi_error.jsonl`** (~1.9MB): Multi-error version with 1,001 samples - some samples contain multiple error spans *(recommended for comprehensive evaluation)*

Both versions are also available via Hugging Face Hub (coming soon).

## 🗂️ Dataset Structure

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
- **`error_spans`**: Tagged version showing error spans
- **`error_type`**: Transformation type (`neg` for negation, `swap` for entity replacement)

### Transformation Types

The dataset contains two types of scientific confabulations:

- **🔄 Negation (`neg`)**: Logical negation of factual statements while preserving plausibility
  - *Example*: "As we get older we **lose** the ability..." → "As we get older we **gain** the ability..."

- **🔀 Entity Replacement (`swap`)**: Substitution of technical terms with plausible but incorrect alternatives  
  - *Example*: "...supported by the **Sodium/Potassium ATPase**..." → "...supported by the **Calcium/Magnesium Pump**..."

### Dataset Versions

**Single-Error Version** (`dataset_single_error.jsonl`):
- 1,251 samples total
- Each sample contains exactly one `<error_type>error_span</error_type>` 
- Simplified evaluation: binary detection + single span localization
- Ideal for baseline evaluation and method development

**Multi-Error Version** (`dataset_multi_error.jsonl`):
- 1,001 samples total  
- 165 samples (16.5%) contain multiple error spans
- Complex evaluation: requires finding ALL errors in a sample
- Better reflects real-world confabulation scenarios where multiple facts may be wrong
- Essential for comprehensive model evaluation

## 🚀 Quick Start

### Loading the Dataset

```python
import json
import pandas as pd

# Load single-error version (recommended for initial evaluation)
with open('dataset_single_error.jsonl', 'r') as f:
    single_error_data = [json.loads(line) for line in f]

# Load multi-error version (for comprehensive evaluation)
with open('dataset_multi_error.jsonl', 'r') as f:
    multi_error_data = [json.loads(line) for line in f]

# Convert to DataFrames
df_single = pd.DataFrame(single_error_data)
df_multi = pd.DataFrame(multi_error_data)

print(f"Single-error dataset: {len(df_single)} samples")
print(f"Multi-error dataset: {len(df_multi)} samples")
print(f"Error types: {df_single['error_type'].value_counts().to_dict()}")
```

### Basic Analysis

```python
# Sample questions
print("Sample questions:")
for i, question in enumerate(df_single['question'].head(3)):
    print(f"{i+1}. {question[:100]}...")

# Compare error distributions
print(f"\nSingle-error distribution:\n{df_single['error_type'].value_counts()}")
print(f"\nMulti-error distribution:\n{df_multi['error_type'].value_counts()}")

# Analyze multi-error complexity
import re
multi_error_counts = []
for error_spans in df_multi['error_spans']:
    count = len(re.findall(r'<(neg|swap)>', error_spans))
    multi_error_counts.append(count)

print(f"\nError span complexity in multi-error version:")
for i in range(1, max(multi_error_counts) + 1):
    count = multi_error_counts.count(i)
    print(f"  {i} error(s): {count} samples")
```

### Using with Hugging Face Datasets

```python
from datasets import load_dataset

# Load single-error version (recommended for initial evaluation)
dataset_single = load_dataset("your-org/refact", "single_error")

# Load multi-error version (for comprehensive evaluation)  
dataset_multi = load_dataset("your-org/refact", "multi_error")

# Access the dataset (single split design)
data_single = dataset_single['train']  # 1,251 samples
data_multi = dataset_multi['train']    # 1,001 samples
```

### Data Splits

ReFACT uses a **single split design** - all samples are provided as training data, allowing researchers to create their own train/validation/test splits based on their experimental needs. This design choice maximizes flexibility for different evaluation scenarios.

## 📊 Evaluation Framework

ReFACT enables evaluation across three progressive tasks:

### 1. 🎯 Confabulation Detection
Binary classification of scientific content as factual or confabulated.

### 2. 🎯 Error Localization  
Identifying specific spans within text that contain factual errors.

### 3. 🎯 Content Correction
Generating corrections to restore factual accuracy.

### Evaluation Baselines

The benchmark results were obtained using zero-shot prompting with task-specific templates. The evaluation framework is designed to be:
- **Independent**: Each task evaluated separately to avoid error compounding.
- **Comprehensive**: Covers binary detection, span localization, and content correction.
- **Robust**: Uses IoU for localization and Exact Match for correction to avoid misleading metrics.


## 🏗️ Dataset Construction

### Source Data
- **Origin**: r/AskScience subreddit (23M+ members)
- **Quality Control**: High-scoring questions and answers only.
- **Filtering**: 500-1000 character length, score ≥4.

### Transformation Pipeline
1. **Fact Extraction**: Identify factual statements using Gemma-2-27B-it.
2. **Transformation**: Apply negation or entity replacement.
3. **Selection**: Choose most convincing confabulated version.
4. **Human Annotation**: Multi-annotator verification (72.5% pairwise agreement rate).

### Quality Assurance
- 3 independent annotators per sample.
- Consensus requirement (≥2/3 agreement).
- Detailed annotation guidelines.
- 58% transformation success rate after iterative refinement.

## 📂 Repository Structure

```
ReFACT/
├── dataset_single_error.jsonl       # Single-error version (1,251 samples)
├── dataset_multi_error.jsonl        # Multi-error version (1,001 samples)
├── create_paraphrased_dataset.py    # Script to generate paraphrased versions  
├── README.md                        # This file  
├── LICENSE                          # MIT license
├── requirements.txt                 # Python dependencies
├── CITATION.cff                     # GitHub citation format
├── ReFACT_Scientific_Confabulation.pdf # Research paper
├── scripts/
│   └── cleanup_dataset.py           # Dataset cleanup utilities
└── huggingface/                     # Hugging Face Hub package
    ├── README.md                    # Dataset card
    ├── refact.py                   # HF Datasets loading script
    └── dataset_infos.json          # HF metadata
```

## ❓ FAQ

**Q: What makes ReFACT different from other factuality benchmarks?**
A: ReFACT focuses specifically on subtle scientific confabulations that require domain expertise to detect, with full human verification and span-level annotations.

**Q: How were the transformations created?**
A: We used a systematic pipeline with LLM-assisted generation followed by rigorous human validation to ensure realistic, challenging confabulations.

**Q: Which dataset version should I use?**
A: Start with `dataset_single_error.jsonl` for initial evaluation and method development. Use `dataset_multi_error.jsonl` for comprehensive evaluation that better reflects real-world scenarios where multiple facts may be incorrect.

**Q: Can I use this for commercial applications?**
A: Yes, the dataset is released under MIT license for both research and commercial use.

## 📜 License

This dataset is released under the [MIT License](LICENSE) (SPDX: MIT). You are free to use, modify, and distribute this dataset for any purpose, including commercial use.

## 📖 Citation

If you use ReFACT in your research, please cite:

```bibtex
@misc{refact2025,
  title={ReFACT: A Benchmark for Scientific Confabulation Detection with Positional Error Annotations},
  author={[Authors]},
  year={2025},
  note={Dataset and benchmark for scientific factuality evaluation},
  url={https://github.com/your-org/refact}
}
```

## 🤝 Contributing

We welcome contributions to improve ReFACT! Please:
- Open an issue for bug reports or feature requests  
- Submit pull requests for improvements
- Follow our code of conduct and contribution guidelines

---