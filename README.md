# A Comparative Evaluation of Hallucination in Large Language Models for English and Hindi

This repository contains a paired-question study of model responses in English and Hindi. The repository contains the complete question bank and the response-collection pipeline used for the study. The response-collection experiment was executed in two batches covering Q001–Q100 and Q101–Q200, corresponding to 1,600 planned model-question-language combinations. The current workspace retains the second batch as an 800-row response workbook; annotation and substantive hallucination analysis are the next stages of the study.

## Research Overview

The study is designed to compare how four large language models respond to the same underlying questions when asked in English and Hindi. It covers factual, numerical, reasoning, and cultural/India-specific questions. Each question has an English and Hindi version, along with an expected answer and source field to support later human evaluation.

The experiment collects raw answers; it does not automatically determine whether an answer is correct or hallucinated. No hallucination rates, model rankings, or comparative findings are reported in this repository.

## Research Objective

To collect comparable English and Hindi responses from four LLMs across four question categories, then use reference-based human annotation to study hallucination behavior by language, category, and model.

## Research Questions

- How does the occurrence and form of unsupported or incorrect content differ between English and Hindi for paired questions?
- How does response quality vary across factual, numerical, reasoning, and cultural/India-specific questions?
- How do responses differ across the four evaluated models?
- Do any language differences vary by question category or model?
- After annotation, are observed English-Hindi differences statistically distinguishable?

These are research questions, not findings. The current analysis functions are placeholders and do not compute answers to them.

## Dataset and Question Bank

The source workbook is `data/raw/question_bank.xlsx`. It contains 200 unique questions, identified Q001–Q200, with 50 questions in each category:

| Category in the question bank | Questions |
|---|---:|
| Factual | 50 |
| Numerical | 50 |
| Reasoning | 50 |
| Cultural/India-specific | 50 |
| **Total** | **200** |

Each row contains an English question, a Hindi question, an expected answer, and a source. The experiment uses both language versions of every question. The workbook was inspected for this README: all 200 IDs are unique, and none of the required question, answer, or source fields are blank.

## Models

The configured models and API identifiers are:

| Model name | Provider | API model ID |
|---|---|---|
| Gemini 3.5 Flash-Lite | Google Gemini | `gemini-3.5-flash-lite` |
| GPT-OSS 120B | Groq | `openai/gpt-oss-120b` |
| Cohere Command A | Cohere | `command-a-03-2025` |
| Qwen 3.8 27B | Groq | `qwen/qwen3.8-27b` |

These are the names and identifiers in the experiment pipeline; provider-side availability and behavior may change over time.

## Experimental Design

Each of the 200 underlying questions has two language versions, and each version is sent to all four models once:

**200 questions × 2 languages × 4 models × 1 run = 1,600 planned model responses.**

The reported execution was split into two batches:

| Batch | Questions | Planned requests |
|---|---|---:|
| 1 | Q001–Q100 | 800 |
| 2 | Q101–Q200 | 800 |
| **Total** | **Q001–Q200** | **1,600** |

The pipeline uses one run per question-language-model combination (`NUM_RUNS = 1`), temperature 0, a maximum of 256 output tokens, and disables external tools and web search.

## Prompting Methodology

The prompt templates in `experiments/prompts.py` use the same neutral system instruction for all models and both languages:

```text
You are an AI assistant taking part in a research study. Respond to the question you are asked.
```

The English user prompt is `Question:` followed by the question and an `Answer:` marker. The Hindi user prompt uses `प्रश्न:` and `उत्तर:` around the Hindi question. The question text changes by language; the system instruction is constant. The prompts do not ask the model to be factual, cite sources, abstain when uncertain, or follow category-specific instructions. Category, expected answer, and source are retained as experiment metadata and are not inserted into the user prompt.

## Experiment Pipeline

1. `experiments/run_experiment.py` reads the Excel question bank with pandas and checks for `Question ID`, `Category`, `English Question`, `Hindi Question`, `Expected Answer`, and `Source` columns. Empty-ID rows are skipped.
2. It creates one prompt record in English and one in Hindi for each selected question, using `experiments/prompts.py`.
3. It initializes one provider client per configured model through the factory in `experiments/model_runner.py`.
4. It sends each prompt to each model. Provider-specific runners call the Gemini, Groq, or Cohere SDK. Requests are retried up to three attempts with exponential delays (2 seconds, then 4 seconds); exhausted exceptions are caught by the experiment loop and recorded as errors for that response.
5. For each attempted response, the pipeline records question and source fields, model/provider identifiers, run number, generation settings, full system and user prompts, response text, status, error text, UTC start and finish timestamps, and latency.
6. It writes the collected rows to `responses/raw_responses/experiment_responses.xlsx` as a single Excel worksheet. A run writes/overwrites this workbook; it does not append to or merge a previous batch automatically.

An error during client initialization occurs before per-request error handling and can stop a run. Error rows in the workbook represent requests that exhausted the configured retries; retries are not separate response rows.

## Output Files and Analysis State

- `data/raw/question_bank.xlsx` — the 200-question English-Hindi question bank.
- `responses/raw_responses/experiment_responses.xlsx` — the generated response workbook currently available in this workspace. It is ignored by Git through `.gitignore` and may not be present in a fresh clone.
- `analysis/hallucination_rates.py`, `analysis/statistical_tests.py`, and `analysis/error_analysis.py` — analysis interfaces/placeholders; their functions raise `NotImplementedError` and produce no results.
- `notebooks/exploratory_analysis.ipynb` — a placeholder notebook; it contains no completed analysis.
- `results/tables/` and `results/figures/` — no generated result tables or figures are currently present.
- `annotations/annotator_1/`, `annotations/annotator_2/`, and `annotations/final_labels/` — no annotation files are currently present. `annotations/annotation_guidelines.md` describes a preliminary framework.

## Current Experiment Status

The 200-question bank is complete, and the full response-collection design was executed in two 100-question batches. Together, these batches cover the complete experimental matrix of 200 questions × 2 languages × 4 models = 1,600 planned model responses. The currently retained workbook contains the second batch, Q101–Q200, with 800 rows. The first batch was executed separately but its original workbook is not currently retained in the repository, so its exact saved success/failure counts cannot be independently verified from the present artifacts.

Counts below therefore describe **the available 800-row workbook only**, not the full 1,600-request experiment:

| Saved response status | Count |
|---|---:|
| Successful | 796 |
| Failed after retries | 4 |
| **Rows in workbook** | **800** |

Of the four saved failures, three GPT-OSS 120B rows report an empty response from Groq after three attempts. One Cohere Command A row records an HTTP 422 `NO_VALID_RESPONSE_GENERATED` error after three attempts. The workbook does not contain successful response text for these four rows. Gemini 3.5 Flash-Lite and Qwen 3.8 27B have 200 successful rows each in this batch; GPT-OSS 120B has 197 successful and 3 failed rows; Cohere Command A has 199 successful and 1 failed row.

The response statuses above indicate whether a model call returned response text; they are **not** correctness or hallucination labels. No human annotation, hallucination scoring, statistical test, or result figure/table has been completed.

## Reproducibility and Setup

Use a Python installation with support for the dependencies in `requirements.txt`. The repository does not pin a Python version.

From the repository root, create and activate a virtual environment, then install dependencies:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Create a local `.env` file in the repository root with the API keys required by the current code:

```dotenv
GEMINI_API_KEY=your_gemini_key
GROQ_API_KEY=your_groq_key
COHERE_API_KEY=your_cohere_key
```

Keep real credentials out of source control. The current `.env.example` contains older variable names; use the names above, which are the ones read by `experiments/model_runner.py`.

Run the experiment from the repository root:

```powershell
python -m experiments.run_experiment
```

All three provider keys are needed because the runner initializes all four configured models. The `RUN_FULL_EXPERIMENT` constant in `experiments/run_experiment.py` controls selection; it is not exposed as a command-line option. When it is `True`, the script selects up to the first 200 questions. In the current code, when it is `False`, it selects `questions[100:200]` (Q101–Q200 for the supplied ordered bank). Although `PILOT_NUM_QUESTIONS` is defined in `experiments/config.py`, this current selection branch does not use it. The script writes to the same output path each time, so separate batch runs are not preserved or combined automatically.

## Next Research Stage

With response collection completed, the next stage is to evaluate the collected model outputs against the reference answers. The planned workflow is:

```text
Raw Model Responses
        ↓
Response Validation
        ↓
Human Annotation
        ↓
Hallucination / Correctness Labels
        ↓
Language-wise Analysis
        ↓
Category-wise Analysis
        ↓
Model-wise Analysis
        ↓
Statistical Analysis
        ↓
Research Findings
```

The analysis will focus on:

- English vs Hindi response behavior
- Model-wise response patterns
- Category-wise differences
- Hallucination and error types
- Qualitative examples
- Statistical comparison of observed differences

No performance ranking or hallucination-rate conclusion is claimed until the annotation and analysis stages are completed.

## Authors

**Diya Pratap** — B.Tech. Artificial Intelligence & Machine Learning, Dr. Akhilesh Das Gupta Institute of Professional Studies, GGSIPU, New Delhi. [GitHub](https://github.com/Diyapratap22)

**Kinjal Sidharth** — B.Tech. Artificial Intelligence & Machine Learning, Dr. Akhilesh Das Gupta Institute of Professional Studies, GGSIPU, New Delhi. [GitHub](https://github.com/Kinjal7127)

**Shivam** — B.Tech. Artificial Intelligence & Machine Learning, Dr. Akhilesh Das Gupta Institute of Professional Studies, GGSIPU, New Delhi. [GitHub](https://github.com/Shivam-po)

## Repository Structure

The following reflects the project files and directories currently present. The generated response workbook is a local, Git-ignored artifact; empty directories are represented by `.gitkeep` files.

```text
.
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── test_api_connections.py
├── analysis/
│   ├── __init__.py
│   ├── error_analysis.py
│   ├── hallucination_rates.py
│   └── statistical_tests.py
├── annotations/
│   ├── annotation_guidelines.md
│   ├── annotator_1/.gitkeep
│   ├── annotator_2/.gitkeep
│   └── final_labels/.gitkeep
├── data/
│   ├── README.md
│   ├── processed/.gitkeep
│   └── raw/
│       ├── .gitkeep
│       └── question_bank.xlsx
├── experiments/
│   ├── __init__.py
│   ├── config.py
│   ├── model_runner.py
│   ├── prompts.py
│   └── run_experiment.py
├── literature/
│   ├── README.md
│   └── literature_matrix.csv
├── notebooks/
│   └── exploratory_analysis.ipynb
├── paper/
│   ├── references.bib
│   └── ieee/.gitkeep
├── responses/
│   ├── README.md
│   └── raw_responses/
│       ├── .gitkeep
│       └── experiment_responses.xlsx  # local output; Git-ignored
└── results/
    ├── figures/.gitkeep
    └── tables/.gitkeep
```