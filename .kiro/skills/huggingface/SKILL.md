# Skill: Hugging Face & Open-Source LLMs

## Purpose

Provides reusable engineering expertise for integrating Hugging Face Transformers and open-source
models into Nexus AI, covering model loading, tokenization, inference pipelines, embedding
generation, license review, and resource-aware deployment.

## When to Use

Activate this skill when:
- Loading a model and tokenizer from Hugging Face Hub
- Running text generation, summarization, or classification via Transformers pipelines
- Generating embeddings with Sentence Transformers
- Wrapping a Hugging Face model behind the `LLMProvider` domain interface
- Evaluating open-source model performance and comparing with hosted providers
- Documenting model licenses and deployment constraints
- Optimizing inference with quantization or device placement

## Core Rules

1. **License review before integration.** Check the model card license before using any model in a product context. Document the license in the implementation file.
2. **Provider abstraction always.** Hugging Face models implement the same `LLMProvider` interface as cloud providers — callers never see `transformers.*` types.
3. **Device placement is explicit.** Always specify `device` or use `device_map="auto"` — never assume GPU availability.
4. **Memory budget is known.** Document VRAM/RAM requirements for each integrated model. Fail fast if the device cannot meet the minimum.
5. **Tokenizer and model are paired.** Load tokenizer and model from the same checkpoint. Do not mix tokenizers across checkpoints.
6. **Streaming requires iteration.** For streaming generation, use `TextIteratorStreamer` in a separate thread — not `.generate()` followed by decode.
7. **Model files are not committed to Git.** Cache to `~/.cache/huggingface` or a configured `HF_HOME` directory. Never commit model weights.

## Project Structure

```
backend/model-serving/huggingface/
├── models/
│   ├── registry.py             — Map of model name → HFModelConfig
│   └── model_config.py         — HFModelConfig dataclass (id, task, license, min_ram_gb)
├── providers/
│   ├── hf_text_generation.py   — Transformers-based LLMProvider
│   └── hf_embedding.py         — Sentence Transformers EmbeddingProvider
├── pipelines/
│   ├── generation_pipeline.py  — Wraps AutoModelForCausalLM + AutoTokenizer
│   └── embedding_pipeline.py   — Wraps SentenceTransformer
├── adapters/
│   └── hf_llm_adapter.py       — Bridges HF provider → domain LLMProvider protocol
└── tests/
    ├── test_hf_provider.py
    └── test_embedding.py
```

## Implementation Patterns

### Model Config Registry

```python
from dataclasses import dataclass

@dataclass
class HFModelConfig:
    model_id: str        # e.g. "microsoft/phi-3-mini-4k-instruct"
    task: str            # "text-generation" | "feature-extraction"
    license: str         # e.g. "MIT", "apache-2.0", "llama3"
    min_ram_gb: float    # minimum system RAM for CPU inference
    min_vram_gb: float   # minimum VRAM for GPU inference

MODEL_REGISTRY: dict[str, HFModelConfig] = {
    "phi-3-mini": HFModelConfig(
        model_id="microsoft/phi-3-mini-4k-instruct",
        task="text-generation",
        license="MIT",
        min_ram_gb=8.0,
        min_vram_gb=4.0,
    ),
    "all-minilm": HFModelConfig(
        model_id="sentence-transformers/all-MiniLM-L6-v2",
        task="feature-extraction",
        license="apache-2.0",
        min_ram_gb=1.0,
        min_vram_gb=0.5,
    ),
}
```

### Text Generation Provider

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, TextIteratorStreamer
from threading import Thread
import torch

class HFTextGenerationProvider:
    """
    License: see MODEL_REGISTRY for the license of the loaded model.
    """

    def __init__(self, model_id: str, device: str = "cpu"):
        self._tokenizer = AutoTokenizer.from_pretrained(model_id)
        self._model = AutoModelForCausalLM.from_pretrained(
            model_id, torch_dtype=torch.float16 if device != "cpu" else torch.float32
        ).to(device)
        self._device = device

    def complete(self, prompt: str, max_new_tokens: int = 256) -> str:
        inputs = self._tokenizer(prompt, return_tensors="pt").to(self._device)
        with torch.no_grad():
            output = self._model.generate(**inputs, max_new_tokens=max_new_tokens)
        return self._tokenizer.decode(output[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True)

    def stream(self, prompt: str, max_new_tokens: int = 256):
        inputs = self._tokenizer(prompt, return_tensors="pt").to(self._device)
        streamer = TextIteratorStreamer(self._tokenizer, skip_special_tokens=True)
        thread = Thread(target=self._model.generate, kwargs={**inputs, "streamer": streamer, "max_new_tokens": max_new_tokens})
        thread.start()
        for token in streamer:
            yield token
        thread.join()
```

### Sentence Transformers Embedding

```python
from sentence_transformers import SentenceTransformer

class HFEmbeddingProvider:
    """
    License: apache-2.0 (all-MiniLM-L6-v2)
    """

    def __init__(self, model_id: str = "sentence-transformers/all-MiniLM-L6-v2"):
        self._model = SentenceTransformer(model_id)

    def embed(self, texts: list[str]) -> list[list[float]]:
        return self._model.encode(texts, normalize_embeddings=True).tolist()
```

## Testing Guidance

- Do not load real models in unit tests — create a `FakeHFProvider` that returns deterministic outputs
- Use a tiny test model (`hf-internal-testing/tiny-random-gpt2`) only in integration tests tagged `@pytest.mark.slow`
- Test streaming by collecting all tokens from the generator and asserting the concatenated output
- Test device placement by mocking `torch.cuda.is_available()` in relevant tests

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Not checking model license | Always check and document license before integration |
| Committing model weight files | Add `*.bin`, `*.safetensors`, `*.pt` to `.gitignore` |
| Using `.generate()` without `max_new_tokens` | Always set a bounded token limit |
| Calling blocking inference in async context | Run in `executor` or use a worker thread |
| Assuming CUDA is available | Check `torch.cuda.is_available()`; fall back gracefully to CPU |
| Mismatched tokenizer/model checkpoint | Load both from the same `model_id` string |

## Relationship to Steering

This skill governs `backend/model-serving/huggingface/`. Hugging Face types must not appear in domain interfaces. Security rules apply: do not log user prompts or generated content containing PII.
