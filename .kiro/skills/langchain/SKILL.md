# Skill: LangChain Integration

## Purpose

Provides reusable engineering expertise for integrating LangChain into the Nexus AI backend,
covering chains, prompt templates, retrievers, tool definitions, document loaders, output parsers,
and the adapter pattern that isolates LangChain internals from domain interfaces.

## When to Use

Activate this skill when:
- Building document Q&A or retrieval-augmented chains
- Creating reusable prompt templates with input variables
- Defining LangChain tools that back application-level tool abstractions
- Implementing document loaders for supported file types
- Using output parsers for structured JSON responses
- Wrapping LangChain components behind an adapter so domain code stays clean
- Writing tests for LangChain-based workflows with fake/mock LLMs

## Core Rules

1. **Adapter pattern always.** Domain interfaces (`LLMProvider`, `Retriever`, `Tool`) are defined in the domain layer. LangChain implementations are adapters in the infrastructure layer. Domain code never imports `langchain.*` directly.
2. **Prompt templates are versioned.** Store templates in files or constants — not inline strings in route handlers.
3. **Chains are composable.** Build chains from small, testable pieces using LCEL (LangChain Expression Language) where available.
4. **LLM is injected.** Never instantiate a `ChatOpenAI` or `ChatGoogleGenerativeAI` inside a chain factory; inject it.
5. **Streaming propagates.** When the underlying LLM streams, the chain must propagate tokens — do not buffer and return at the end.
6. **Metadata is preserved.** Document loaders must preserve source, page number, and metadata through the retrieval chain.

## Project Structure

```
backend/agent-runtime/langchain/
├── chains/
│   ├── chat_chain.py           — Basic chat with memory
│   ├── rag_chain.py            — RAG retrieval + generation
│   └── document_qa_chain.py    — Document question answering
├── prompts/
│   ├── system_prompts.py
│   └── templates/
│       ├── rag_template.txt
│       └── qa_template.txt
├── tools/
│   ├── search_tool.py
│   └── calculator_tool.py
├── loaders/
│   ├── pdf_loader.py
│   └── text_loader.py
├── adapters/
│   ├── langchain_llm_adapter.py   — wraps ChatOpenAI → LLMProvider
│   └── langchain_retriever_adapter.py
└── tests/
    ├── test_rag_chain.py
    └── test_chat_chain.py
```

## Implementation Patterns

### LCEL RAG Chain

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_core.language_models import BaseChatModel
from langchain_core.vectorstores import VectorStoreRetriever

RAG_TEMPLATE = """You are a helpful assistant. Answer using only the context below.
If the answer is not in the context, say you don't know.

Context:
{context}

Question: {question}
"""

def build_rag_chain(llm: BaseChatModel, retriever: VectorStoreRetriever):
    prompt = ChatPromptTemplate.from_template(RAG_TEMPLATE)
    return (
        {"context": retriever, "question": RunnablePassthrough()}
        | prompt
        | llm
        | StrOutputParser()
    )
```

### LangChain Tool Definition

```python
from langchain_core.tools import tool

@tool
def search_documents(query: str) -> str:
    """Search the document store for information relevant to the query."""
    # implementation calls domain retrieval service
    ...
```

### Adapter: LangChain LLM → Domain Protocol

```python
from langchain_core.language_models import BaseChatModel
from langchain_core.messages import HumanMessage
from domain.interfaces.llm_provider import LLMProvider, ChatRequest, ChatResponse

class LangChainLLMAdapter(LLMProvider):
    def __init__(self, llm: BaseChatModel):
        self._llm = llm

    async def complete(self, request: ChatRequest) -> ChatResponse:
        messages = [HumanMessage(content=m.content) for m in request.messages]
        result = await self._llm.ainvoke(messages)
        return ChatResponse(content=result.content, model=request.model)
```

### Fake LLM for Tests

```python
from langchain_core.language_models.fake import FakeListChatModel

def make_fake_llm(responses: list[str]) -> FakeListChatModel:
    return FakeListChatModel(responses=responses)
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Importing `langchain.*` in domain layer | Always use adapters; domain imports only domain interfaces |
| Instantiating `ChatOpenAI(api_key=...)` inside chain | Inject the LLM; read key from settings |
| Blocking `.invoke()` in async context | Use `.ainvoke()` for async chains |
| Ignoring document metadata | Pass `metadata` through loader and preserve in retrieved chunks |
| Storing prompt strings inline in handlers | Centralize in `prompts/` module |
| Testing against real LLM APIs | Use `FakeListChatModel` or `FakeChatModel` |

## Testing Guidance

- Use `FakeListChatModel` for deterministic chain outputs
- Test each chain stage independently: retriever mock → prompt rendering → output parsing
- Verify that retrieved document metadata (source, page) appears in the response
- Use `pytest-asyncio` for async chain tests
- Test streaming by collecting from `astream()` and asserting token sequence

## Relationship to Steering

This skill governs `backend/agent-runtime/langchain/`. LangChain types must never appear in domain interfaces or the Android codebase. If this skill's guidance conflicts with `05-security-standards.md`, security takes precedence.
