# Skill: LangGraph Workflows

## Purpose

Provides reusable engineering expertise for building stateful AI workflows in LangGraph,
covering graph state definition, node functions, conditional routing, checkpointing,
human-in-the-loop approval, and resumable execution patterns.

## When to Use

Activate this skill when:
- Designing multi-step AI workflows that require branching or looping
- Implementing research, planning, or document-processing workflows
- Adding checkpoints so a workflow can be paused and resumed
- Implementing human approval gates before sensitive tool execution
- Wiring conditional routing between nodes based on graph state
- Testing graph execution with controlled inputs and state inspection

## Core Rules

1. **State is a typed TypedDict.** Every graph has an explicit `GraphState` typed dict — no ad-hoc dicts.
2. **Nodes are pure functions.** Each node receives the current state and returns a partial state update — no side effects outside the update.
3. **Edges express routing logic.** Use conditional edges (`add_conditional_edges`) for branching; avoid putting routing logic inside nodes.
4. **Checkpoints are explicit.** Attach a `MemorySaver` or `SqliteSaver` checkpoint when resumability is required.
5. **Step limits are enforced.** Every graph that can loop must have a maximum iteration counter in the state and a guard edge.
6. **Human approval is a node.** Implement human-in-the-loop as an `interrupt_before` or dedicated approval node — not as a blocking call inside a tool.
7. **LLM is injected.** Never instantiate the LLM inside a node function; inject it via a closure or dependency.

## Project Structure

```
backend/agent-runtime/langgraph/
├── graphs/
│   ├── research_graph.py      — Multi-step research workflow
│   ├── document_graph.py      — Document analysis workflow
│   └── approval_graph.py      — Human-approval workflow
├── nodes/
│   ├── planner.py
│   ├── researcher.py
│   ├── summarizer.py
│   └── human_review.py
├── state/
│   └── graph_state.py         — Shared TypedDict definitions
├── checkpoints/
│   └── checkpoint_factory.py  — MemorySaver / SqliteSaver factory
└── tests/
    ├── test_research_graph.py
    └── test_routing.py
```

## Implementation Patterns

### Graph State

```python
from typing import TypedDict, Annotated
from langchain_core.messages import BaseMessage
import operator

class ResearchState(TypedDict):
    query: str
    messages: Annotated[list[BaseMessage], operator.add]
    documents: list[str]
    summary: str
    step_count: int
    requires_approval: bool
    approved: bool
```

### Node Function

```python
from langchain_core.language_models import BaseChatModel
from state.graph_state import ResearchState

def make_researcher_node(llm: BaseChatModel):
    async def researcher(state: ResearchState) -> dict:
        response = await llm.ainvoke(state["messages"])
        return {
            "messages": [response],
            "step_count": state["step_count"] + 1,
        }
    return researcher
```

### Conditional Routing

```python
from state.graph_state import ResearchState

def should_continue(state: ResearchState) -> str:
    if state["step_count"] >= 10:
        return "end"
    if state.get("requires_approval") and not state.get("approved"):
        return "human_review"
    if state.get("summary"):
        return "end"
    return "researcher"
```

### Graph Assembly

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from state.graph_state import ResearchState

def build_research_graph(llm):
    researcher = make_researcher_node(llm)
    summarizer = make_summarizer_node(llm)

    graph = StateGraph(ResearchState)
    graph.add_node("researcher", researcher)
    graph.add_node("summarizer", summarizer)
    graph.add_node("human_review", human_review_node)

    graph.set_entry_point("researcher")
    graph.add_conditional_edges("researcher", should_continue, {
        "researcher": "researcher",
        "human_review": "human_review",
        "end": "summarizer",
    })
    graph.add_edge("human_review", "researcher")
    graph.add_edge("summarizer", END)

    checkpointer = MemorySaver()
    return graph.compile(checkpointer=checkpointer)
```

### Running with Thread ID (resumable)

```python
config = {"configurable": {"thread_id": "research-session-1"}}
result = await app.ainvoke({"query": "What is RAG?", "step_count": 0}, config=config)
```

## Testing Guidance

- Test individual node functions in isolation with a fake LLM
- Test routing functions with hand-crafted state dictionaries — no graph needed
- Test the full graph with `FakeListChatModel` and assert final state fields
- Test step-limit enforcement by running the graph with a mock that never produces a `summary`
- Test checkpointing by running the graph to a midpoint, inspecting state, then resuming

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Routing logic inside node function | Use `add_conditional_edges` with a routing function |
| Mutable default in TypedDict | Use `Annotated[list, operator.add]` for list accumulation |
| Unbounded loops | Add `step_count` to state and a guard edge |
| Blocking calls in async nodes | Use `await` for all LLM and tool calls |
| Hardcoding thread_id | Pass thread_id from the caller; use UUID for new sessions |

## Relationship to Steering

This skill governs `backend/agent-runtime/langgraph/`. LangGraph types must not appear in Android code or domain interfaces. `05-security-standards.md` applies — validate all tool inputs before execution inside nodes.
