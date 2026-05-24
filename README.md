# LLM Output Analyser

An end-to-end pipeline for evaluating Large Language Model outputs
at scale. Evaluates GPT-3.5-turbo across 30 questions using a
multi-dimensional scoring rubric, classifies failure modes, and
produces a structured JSON report with recommendations.

---

## What it does

1. **Designs 30 evaluation questions** across 3 expert domains
2. **Collects LLM responses** via OpenAI GPT-3.5-turbo API
3. **Scores each response** on a 4-dimension weighted rubric
4. **Classifies failure modes** for responses that fall short
5. **Produces a structured JSON report** with actionable recommendations

---

## Question domains

| Domain | Questions | Topics covered |
|--------|-----------|----------------|
| Economics | 10 | GDP, inflation, unemployment, fiscal/monetary policy, Gini coefficient, PPP, Okun's Law |
| Data Analysis | 10 | Correlation vs causation, precision/recall, overfitting, MAE vs RMSE, data leakage, class imbalance |
| Research & Evaluation | 10 | RCTs, selection bias, internal/external validity, MEAL framework, p-hacking, inter-rater reliability |

Domain expertise is the key differentiator for high-quality AI evaluation
work. These domains were chosen because they represent Abigael's genuine
subject matter knowledge — enabling reliable judgement of model correctness.

---

## Scoring rubric

Each response is scored on 4 dimensions with weighted importance:

| Dimension | Weight | What it measures |
|-----------|--------|-----------------|
| Concept coverage | 40% | Are the expected key concepts present in the response? |
| Accuracy signal | 30% | Are there factual errors or obvious contradictions? |
| Clarity | 15% | Is the response well structured and easy to follow? |
| Completeness | 15% | Does it fully address the question asked? |

**Pass threshold: 0.7** total weighted score

---

## Results — GPT-3.5-turbo

![Evaluation Results](llm_evaluation_results.png)

### Overall

| Metric | Result |
|--------|--------|
| Overall pass rate | **96.7%** |
| Average score | **0.8307** |
| Questions passed | 29 / 30 |
| Questions failed | 1 / 30 |
| Run timestamp | 2026-05-24 19:37:14 |

### By domain

| Domain | Pass rate | Avg score | n |
|--------|-----------|-----------|---|
| Economics | 100% | 0.892 | 10 |
| Data Analysis | 100% | 0.800 | 10 |
| Research & Evaluation | 90% | 0.800 | 10 |

### By difficulty

| Difficulty | Pass rate | Avg score | n |
|------------|-----------|-----------|---|
| Easy | 100% | 0.860 | 10 |
| Medium | 94.1% | 0.800 | 17 |
| Hard | 100% | 0.907 | 3 |

### Rubric dimension scores

| Dimension | Score | Status |
|-----------|-------|--------|
| Concept coverage | 0.577 | ⚠ Below threshold — needs improvement |
| Accuracy signal | 1.000 | ✓ Perfect |
| Clarity | 1.000 | ✓ Perfect |
| Completeness | 1.000 | ✓ Perfect |

### Failure mode breakdown

| Mode | Count |
|------|-------|
| Pass | 29 |
| Missing concepts | 1 |

---

## Key evaluation findings

**Finding 1 — GPT-3.5-turbo is accurate and clear but misses specific vocabulary**

The model scored 1.0 on accuracy, clarity, and completeness across all 30
questions. However concept coverage scored only 0.577 — the model answers
correctly without always using the specific technical terminology the rubric
requires. For example it may explain the Phillips Curve accurately without
explicitly using the words "stagflation" or "trade-off."

This is a well-documented phenomenon in LLM evaluation: a model can be
factually correct while failing terminology-based rubrics. This pipeline
surfaces that gap explicitly.

**Finding 2 — Hard questions outperformed Easy and Medium ones**

Hard questions averaged 0.907 versus 0.860 for Easy and 0.800 for Medium.
The likely explanation: Hard questions are more specific and technical,
giving the model clearer vocabulary to anchor its response on. Easy
questions sometimes produce vaguer answers that miss expected terminology
even when the core answer is correct.

**Finding 3 — The single failure is in the Research domain**

One Medium difficulty Research question scored below 0.7 due to missing
concepts. The response was well written and complete but omitted specific
terms the rubric expected — correctly classified as `missing_concepts`
rather than `incomplete_answer` or `clarity_issue`.

**Finding 4 — Economics is the strongest domain**

Economics scored 0.892 average with 100% pass rate — the highest of
all three domains. GPT-3.5-turbo handles macroeconomics concepts
confidently, likely due to high coverage of economic text in training data.

---

## Recommendations

1. **Refine prompts for concept coverage** — explicitly instruct the model
   to use domain-specific terminology in its responses. This addresses the
   0.577 concept score directly.

2. **Re-run the failed question at temperature=0** — determine whether the
   Research failure is a consistent model weakness or a one-off sampling
   artefact.

3. **Extend to GPT-4 comparison** — re-run the same 30 questions with
   GPT-4 to measure the performance gap, particularly on Hard questions
   and concept coverage.

4. **Add adversarial questions** — include questions designed to elicit
   incorrect answers to test whether the model confidently produces errors
   in edge cases.

5. **Increase to 50+ questions per domain** for production-grade evaluation
   with statistically meaningful domain-level comparisons.

---

## Failure mode taxonomy

| Failure mode | Description |
|-------------|-------------|
| `pass` | Response meets the 0.7 quality threshold |
| `missing_concepts` | Factually reasonable but omits critical technical concepts |
| `incomplete_answer` | Response too short or missing key points |
| `clarity_issue` | Hard to follow or poorly structured |
| `api_error` | Response not collected due to API failure |

---

## How to evaluate a different model

Replace the `get_llm_response()` function with your own API call:

```python
def get_llm_response(question_text, client):
    # Replace with your model's API call
    response = your_model.generate(
        prompt=question_text,
        max_tokens=500
    )
    return {
        'response'     : response.text,
        'input_tokens' : response.usage.input,
        'output_tokens': response.usage.output,
        'error'        : None
    }
```

All scoring, failure mode classification, and reporting run
automatically on whatever responses you provide.

---

## How to add your own question domains

```python
custom_questions = [
    {
        'id'          : 'CUSTOM-Q01',
        'question'    : 'Your question here',
        'difficulty'  : 'Medium',
        'key_concepts': ['concept1', 'concept2', 'concept3']
    }
]

all_questions = all_questions + custom_questions
```

---

## Output files

| File | Description |
|------|-------------|
| `llm_evaluation_report.json` | Full structured report with all scores, domain breakdowns, failure modes and recommendations |
| `llm_evaluation_results.png` | 4-chart dashboard: score distribution, domain comparison, rubric dimensions, failure modes |

---

## Skills demonstrated

- OpenAI API integration with error handling and rate limit protection
- Multi-dimensional rubric design with weighted scoring
- Failure mode taxonomy and automated classification
- Structured JSON report engineering for downstream pipeline consumption
- Domain expertise in economics, data analysis, and research methodology
- End-to-end evaluation pipeline design

---

## Project context

This is Project 4 of a 4-project AI evaluation engineering portfolio:

| Project | Repo | What it demonstrates |
|---------|------|---------------------|
| 1 | ml-output-evaluator | Classification and regression metrics, threshold sensitivity |
| 2 | data-quality-validator | Automated data validation pipeline, pytest tests |
| 3 | benchmark-task-suite | 20 benchmark tasks with automated scoring functions |
| 4 | llm-eval-analysis | End-to-end LLM evaluation pipeline (this repo) |

---

## Author

**Abigael Cherotich** — Data Analyst & AI Evaluation Specialist

Nairobi, Kenya | [LinkedIn]([https://www.linkedin.com/in/abigael-cherotich/)
