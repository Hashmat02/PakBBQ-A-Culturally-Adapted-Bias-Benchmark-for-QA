# PakBBQ: A Culturally Adapted Bias Benchmark for QA

[![Paper](https://img.shields.io/badge/ACL%20Anthology-2025.emnlp--main.818-b31b1b)](https://aclanthology.org/2025.emnlp-main.818.pdf)

**Abdullah Hashmat, Muhammad Arham Mirza, Agha Ali Raza** — Lahore University of Management Sciences (LUMS)

Accepted at **EMNLP 2025 (Main Conference)**.

PakBBQ is a culturally and regionally adapted extension of the [BBQ](https://github.com/nyu-mll/BBQ) dataset for measuring social bias in LLMs in the Pakistani context. It contains **214 templates** instantiated into **17,180 QA pairs** across **8 bias categories** (Age, Disability Status, Gender Identity, Physical Appearance, Regional, Religion, SES, Language Formality), in both **English and Urdu**.

Full details on dataset construction, methodology, and results are in the paper: https://aclanthology.org/2025.emnlp-main.818.pdf

## Repository Structure

```
.
├── PakBBQ/
│   ├── PakBBQ_ENG/            # English dataset, one JSONL per bias category
│   └── PakBBQ_UR/             # Urdu dataset, one JSONL per bias category
├── GPT_and_DeepSeek_eval.py/
│   └── streamline_batch_pipeline.py   # Batch evaluation pipeline for OpenAI + DeepSeek models
├── Gemini_eval.py/
│   ├── create_jsonl.py        # Build Gemini batch input from PakBBQ
│   ├── submit_jobs.py         # Submit batch job to Vertex AI / Gemini
│   ├── extract_answers.py     # Parse model responses (incl. Urdu script options)
│   └── mapping.py             # Score responses against ground-truth labels
├── Analysis/                  # Per-model results, metrics and plotting scripts
└── requirements.txt
```

## Environment Setup

```bash
pip install -r requirements.txt
```

## Evaluation Setup for OpenAI/DeepSeek Models

| Parameter      | Required | Description                                                                                          |
| -------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| `--model`      |   Yes    | Name of the model to evaluate (e.g., `gpt-4`, `gpt-4.1-mini`, `gpt-4.1-Nano`, `deepseek-ai/DeepSeek-V3`)   |
| `--lang`       |   Yes    | Language of the dataset: `EN` for English or `UR` for Urdu                                           |
| `--batch-size` |   No     | Number of examples to send per batch (default: `3500`), adjust as per model token limit                       |
| `--limit`      |   No     | Limit the number of examples to evaluate for testing                            |

1. Run the following command in the CLI.
    ```bash
    cd GPT_and_DeepSeek_eval.py

    python3 streamline_batch_pipeline.py \
      --model <Model_Name> \
      --lang <EN/UR> \
      --batch-size <3500> 
    ```

2. To test a subset of data first.
    ```bash
    cd GPT_and_DeepSeek_eval.py

    python3 streamline_batch_pipeline.py \
      --model <Model_Name> \
      --lang <EN/UR> \
      --batch-size <3500> \
      --limit <5>
    ```

Final results will be generated in a ModelName_Lang_Results.csv file, which can then be used to calculate any relevant metrics.

## Evaluation with Google Gemini

This repository contains scripts to evaluate biases in large language models (LLMs) using the PakBBQ dataset with Google's Gemini model. The pipeline processes English and Urdu question-answering data, submits batch jobs to Gemini, extracts responses, and evaluates results.

### Prerequisites

- Python 3.8+
- Google Cloud account with API key
- Installed dependencies: `google-genai`, `pandas`, `python-dotenv`

### Usage

The pipeline consists of four scripts, each with CLI arguments for customization.

1. **Create JSONL Input**:
   ```bash
   python create_jsonl.py --data-folder ../PakBBQ/data --language english --output-file gemini_input.jsonl
   ```
   For Urdu:
   ```bash
   python create_jsonl.py --data-folder ../PakBBQ/data_Urdu --language urdu
   ```

2. **Submit Batch Job**:
   ```bash
   python submit_jobs.py --key-path /path/to/client_secret.json --input-gcs-uri gs://your-bucket/input.jsonl --output-gcs-uri gs://your-bucket/output/
   ```

3. **Extract Answers**:
   ```bash
   python extract_answers.py --results-file responses_gemini-2.0-flash-lite.jsonl --language english
   ```
   For Urdu:
   ```bash
   python extract_answers.py --language urdu
   ```

4. **Evaluate Results**:
   ```bash
   python mapping.py --data-folder ../PakBBQ/data --model-answers-file model_answers.json --output-file results.csv
   ```

### Files

- `create_jsonl.py`: Generates JSONL input for Gemini from PakBBQ data (English or Urdu).
- `submit_jobs.py`: Submits batch jobs to Gemini via Google Cloud.
- `extract_answers.py`: Extracts model responses and maps Urdu answers to English (A, B, C).
- `mapping.py`: Matches responses to ground truth and saves evaluation results as CSV.

### Notes

- Urdu processing requires UTF-8 encoding support.
- Use `--help` with any script for CLI argument details (e.g., `python create_jsonl.py --help`).

## Citation

```bibtex
@inproceedings{hashmat-etal-2025-pakbbq,
    title     = "{P}ak{BBQ}: A Culturally Adapted Bias Benchmark for {QA}",
    author    = "Hashmat, Abdullah and Mirza, Muhammad Arham and Raza, Agha Ali",
    booktitle = "Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing",
    month     = nov,
    year      = "2025",
    address   = "Suzhou, China",
    publisher = "Association for Computational Linguistics",
    pages     = "16160--16172",
    url       = "https://aclanthology.org/2025.emnlp-main.818/"
}
```

## Responsible Use

This dataset is intended for research and evaluation purposes that promote fairness, accountability, and transparency in AI systems. Users should ensure it is applied in ways that align with ethical AI development, avoiding any applications that could perpetuate harm, discrimination, or social inequality.
