# Skill: AI Evaluation

## Purpose

Provides reusable engineering expertise for evaluating Nexus AI components: RAG pipelines,
LLM response quality, agent task completion, and operational metrics (latency, cost, token usage).

## When to Use

Activate this skill when:
- Building evaluation datasets for RAG or agent workflows
- Measuring answer quality, groundedness, or faithfulness
- Comparing model versions, prompt templates, or retrieval configurations
- Tracking token usage, latency, and per-request cost
- Generating evaluation reports for the AI dashboard
- Selecting evaluation frameworks (RAGAS, LangSmith, custom)

## Core Rules

1. **Evaluation datasets are versioned.** Store evaluation sets in `backend/evaluation/datasets/` as JSON or CSV with schema documentation. Never overwrite — append versioned files.
2. **Ground truth is required for quality metrics.** Faithfulness, relevance, and answer correctness require reference answers. Document how ground truth was created.
3. **Metrics are reproducible.** Given the same dataset and model version, evaluation must produce the same scores. Pin model versions and temperatures.
4. **Latency and cost are first-class metrics.** Track `response_time_ms`, `prompt_tokens`, `completion_tokens`, and estimated `cost_usd` for every evaluation run.
5. **Never use production user data as evaluation data without explicit consent and anonymization.**
6. **Evaluation runs are logged.** Each run produces a report with: date, model version, dataset version, all metric scores, and summary statistics.

## Metric Taxonomy

### RAG Evaluation Metrics

| Metric | Description | Range |
|---|---|---|
| Answer Faithfulness | Is the answer supported by retrieved context? | 0–1 |
| Answer Relevance | Does the answer address the question? | 0–1 |
| Context Precision | Are retrieved chunks relevant to the question? | 0–1 |
| Context Recall | Are all relevant chunks retrieved? | 0–1 |

### Agent Evaluation Metrics

| Metric | Description |
|---|---|
| Task Completion Rate | % of tasks completed successfully |
| Steps to Completion | Average steps taken |
| Tool Call Accuracy | % of tool calls that were valid and useful |
| Step Limit Hit Rate | % of runs that hit the maximum step limit |

### Operational Metrics

| Metric | Unit |
|---|---|
| Response Time | milliseconds (p50, p95, p99) |
| Prompt Tokens | count per request |
| Completion Tokens | count per request |
| Estimated Cost | USD per request |

## Project Structure

```
backend/evaluation/
├── datasets/
│   ├── rag_eval_v1.json         — Q&A pairs with reference answers
│   ├── agent_eval_v1.json       — Task definitions with success criteria
│   └── schema/
│       ├── rag_dataset_schema.json
│       └── agent_dataset_schema.json
├── metrics/
│   ├── rag_metrics.py           — Faithfulness, relevance, precision, recall
│   ├── agent_metrics.py         — Completion rate, steps, tool accuracy
│   └── operational_metrics.py   — Latency, tokens, cost
├── runners/
│   ├── rag_evaluator.py         — Runs RAG eval against dataset
│   └── agent_evaluator.py       — Runs agent eval against dataset
├── reports/
│   └── report_generator.py      — Generates JSON/HTML eval report
└── tests/
    └── test_metrics.py
```

## Implementation Patterns

### RAG Dataset Schema

```json
{
  "version": "1.0",
  "created": "2026-10-01",
  "description": "Nexus AI RAG evaluation set — general knowledge documents",
  "items": [
    {
      "id": "q001",
      "question": "What is retrieval-augmented generation?",
      "reference_answer": "RAG is a technique that retrieves relevant documents and injects them into an LLM prompt to improve accuracy.",
      "relevant_document_ids": ["doc_001", "doc_002"]
    }
  ]
}
```

### RAGAS-style Faithfulness Metric

```python
from dataclasses import dataclass

@dataclass
class RAGEvalResult:
    question: str
    answer: str
    context: list[str]
    faithfulness: float       # 0–1: answer supported by context
    answer_relevance: float   # 0–1: answer addresses question
    context_precision: float  # 0–1: retrieved chunks were relevant
    context_recall: float     # 0–1: relevant chunks were retrieved

async def evaluate_faithfulness(
    question: str,
    answer: str,
    context: list[str],
    judge_llm,
) -> float:
    """
    Ask a judge LLM to verify each claim in the answer against the context.
    Returns proportion of claims supported by context.
    """
    prompt = f"""Given the context below, evaluate if the answer is factually supported.
Context: {chr(10).join(context)}
Answer: {answer}
Return a JSON: {{"supported_claims": N, "total_claims": M}}"""
    result = await judge_llm.complete(prompt)
    parsed = parse_json_safely(result)
    total = parsed.get("total_claims", 1)
    return parsed.get("supported_claims", 0) / total if total > 0 else 0.0
```

### Evaluation Run Report

```python
from datetime import datetime, timezone
from dataclasses import dataclass, field

@dataclass
class EvaluationReport:
    run_id: str
    timestamp: str = field(default_factory=lambda: datetime.now(timezone.utc).isoformat())
    model_version: str = ""
    dataset_version: str = ""
    metrics: dict[str, float] = field(default_factory=dict)
    operational: dict[str, float] = field(default_factory=dict)
    num_samples: int = 0
    notes: str = ""
```

## Testing Guidance

- Unit-test metric calculations with hand-crafted inputs and known expected scores
- Test report generation with a minimal `EvaluationReport` to verify output format
- Use a `FakeLLM` judge for faithfulness tests — never call real APIs
- Test dataset loading by parsing schema validation against `rag_eval_v1.json`
- Verify that a report is generated even when some metric computations fail (graceful partial output)

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Using production user data without anonymization | Create synthetic or consented datasets |
| Overwriting previous evaluation datasets | Append versioned files; never overwrite |
| Floating-point metric accumulation error | Use `statistics.mean()` not manual sum/count |
| Not pinning model version for eval runs | Store model name + version in `EvaluationReport` |
| Running judge LLM with high temperature | Use `temperature=0` for reproducible judge outputs |

## Relationship to Steering

This skill governs `backend/evaluation/`. Evaluation results must never contain raw user PII. `05-security-standards.md` applies to evaluation data handling.
