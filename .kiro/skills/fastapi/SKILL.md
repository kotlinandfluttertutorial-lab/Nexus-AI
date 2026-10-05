# Skill: FastAPI LLM Service

## Purpose

Provides reusable engineering expertise for building the Nexus AI Python backend using FastAPI,
including typed request/response models, provider service integration, authentication middleware,
streaming endpoints, and service-level tests.

## When to Use

Activate this skill when:
- Creating or modifying any FastAPI route, router, or endpoint
- Defining Pydantic request/response models for the LLM API
- Implementing provider service classes (OpenAI, Gemini, Ollama wrappers)
- Adding authentication or API-key middleware
- Implementing streaming responses with Server-Sent Events
- Writing pytest-based service tests or httpx integration tests
- Configuring application settings via `pydantic-settings`

## Core Rules

1. **Typed everywhere.** Every route has typed Pydantic request and response models.
2. **Dependency injection via FastAPI `Depends`.** Never instantiate services inline in route handlers.
3. **Async routes for IO.** All routes that call LLM providers or databases must be `async def`.
4. **Provider abstraction.** Route handlers call a provider service interface — never the SDK directly.
5. **Streaming with SSE.** Use `StreamingResponse` with `text/event-stream` for token-by-token delivery.
6. **Settings via environment.** All configuration (`API_KEY`, model names, URLs) comes from `pydantic-settings` / environment variables — never hardcoded.
7. **Structured errors.** Return `HTTPException` with a consistent `{"detail": {"code": ..., "message": ...}}` shape.

## Project Structure

```
backend/fastapi-service/
├── api/
│   ├── routers/
│   │   ├── chat.py          — /v1/chat/completions
│   │   ├── embeddings.py    — /v1/embeddings
│   │   ├── health.py        — /health, /readiness
│   │   └── models.py        — /v1/models
│   └── middleware/
│       └── auth.py          — API-key validation
├── application/
│   └── services/
│       ├── chat_service.py
│       └── embedding_service.py
├── domain/
│   ├── models/
│   │   ├── chat.py          — ChatRequest, ChatResponse, StreamChunk
│   │   └── embeddings.py    — EmbeddingRequest, EmbeddingResponse
│   └── interfaces/
│       └── llm_provider.py  — LLMProvider protocol
├── infrastructure/
│   ├── providers/
│   │   ├── openai_provider.py
│   │   ├── gemini_provider.py
│   │   └── ollama_provider.py
│   └── config.py            — pydantic-settings Settings class
├── tests/
│   ├── unit/
│   └── integration/
└── main.py
```

## Implementation Patterns

### Settings

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    openai_api_key: str = ""
    gemini_api_key: str = ""
    ollama_base_url: str = "http://localhost:11434"
    default_provider: str = "openai"
    default_model: str = "gpt-4o-mini"
    api_key_header: str = "X-API-Key"
    app_api_key: str = ""

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()
```

### LLM Provider Protocol

```python
from typing import Protocol, AsyncIterator
from domain.models.chat import ChatRequest, ChatResponse, StreamChunk

class LLMProvider(Protocol):
    async def complete(self, request: ChatRequest) -> ChatResponse: ...
    async def stream(self, request: ChatRequest) -> AsyncIterator[StreamChunk]: ...
```

### Chat Router with Streaming

```python
from fastapi import APIRouter, Depends
from fastapi.responses import StreamingResponse
from domain.models.chat import ChatRequest
from application.services.chat_service import ChatService
import json

router = APIRouter(prefix="/v1/chat", tags=["chat"])

@router.post("/completions")
async def chat_completions(
    request: ChatRequest,
    service: ChatService = Depends(),
):
    if request.stream:
        async def event_stream():
            async for chunk in service.stream(request):
                yield f"data: {chunk.model_dump_json()}\n\n"
            yield "data: [DONE]\n\n"
        return StreamingResponse(event_stream(), media_type="text/event-stream")
    return await service.complete(request)
```

### API Key Middleware

```python
from fastapi import Security, HTTPException, status
from fastapi.security.api_key import APIKeyHeader
from infrastructure.config import settings

api_key_header = APIKeyHeader(name="X-API-Key", auto_error=False)

async def verify_api_key(api_key: str = Security(api_key_header)):
    if not settings.app_api_key or api_key != settings.app_api_key:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid API key")
    return api_key
```

### Pydantic Models

```python
from pydantic import BaseModel, Field
from typing import Literal

class Message(BaseModel):
    role: Literal["system", "user", "assistant"]
    content: str

class ChatRequest(BaseModel):
    model: str = Field(default="gpt-4o-mini")
    messages: list[Message]
    temperature: float = Field(default=0.7, ge=0.0, le=2.0)
    max_tokens: int = Field(default=1024, ge=1, le=8192)
    stream: bool = False

class ChatResponse(BaseModel):
    id: str
    model: str
    content: str
    usage: dict[str, int]
```

### pytest Integration Test

```python
from httpx import AsyncClient
import pytest

@pytest.mark.asyncio
async def test_chat_completions(async_client: AsyncClient):
    response = await async_client.post(
        "/v1/chat/completions",
        json={"messages": [{"role": "user", "content": "Hello"}]},
        headers={"X-API-Key": "test-key"},
    )
    assert response.status_code == 200
    data = response.json()
    assert "content" in data
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Calling OpenAI SDK directly in a route | Inject a provider via `Depends` |
| Hardcoding API keys in source | Use `pydantic-settings` / `.env` |
| Blocking calls inside `async def` | Use `await` for all IO; use `run_in_executor` for sync SDK calls |
| Missing error handling in streaming | Wrap stream generator in try/except; yield error event |
| Returning raw provider exceptions | Map to `HTTPException` with domain error codes |
| Skipping response model typing | Always declare `response_model` on route decorators |

## Testing Guidance

- Use `httpx.AsyncClient` with `app` override for integration tests
- Use a `FakeLLMProvider` that returns deterministic responses — never call real APIs in tests
- Test streaming by collecting all SSE events and verifying the sequence
- Use `pytest-asyncio` with `asyncio_mode = "auto"` in `pytest.ini`
- Keep `.env.test` with fake/mock credentials separate from `.env`

## Relationship to Steering

This skill applies to `backend/fastapi-service/`. Android architecture steering rules (`01-architecture.md`, `02-android-development.md`) do not govern the Python backend. Security standards (`05-security-standards.md`) apply to both workstreams.
