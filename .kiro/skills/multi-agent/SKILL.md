# Skill: Multi-Agent Systems

## Purpose

Provides reusable engineering expertise for building multi-agent architectures in Nexus AI,
covering supervisor patterns, specialist agent definitions, shared tool registries,
inter-agent communication, task delegation, result aggregation, and safeguards.

## When to Use

Activate this skill when:
- Designing a system where multiple specialist agents collaborate on a task
- Implementing a supervisor or coordinator that delegates to sub-agents
- Sharing a tool registry across multiple agents
- Defining inter-agent message passing and state handoff
- Enforcing step limits, timeout, and loop prevention across an agent network
- Testing multi-agent workflows with fake agents and fake tools

## Core Rules

1. **Supervisor coordinates; specialists execute.** The supervisor decides which specialist handles each sub-task — it does not execute tools directly.
2. **Agents communicate through shared state or messages, not direct method calls.** Use a shared context object or message bus.
3. **Every agent has a maximum step limit.** Enforce independently per agent and globally across the workflow.
4. **Shared tools are registered once.** Maintain a single `ToolRegistry`; agents look up tools by name — no agent owns a tool privately.
5. **Agents never call other agents directly.** All delegation goes through the supervisor.
6. **Cancellation propagates.** When the top-level workflow is cancelled, all running sub-agents must be cancelled.
7. **Results are typed.** Every agent returns a typed `AgentResult` — not a raw string.

## Architecture

```
User Task
    ↓
Supervisor Agent
    ├── delegates to → Researcher Agent
    ├── delegates to → Writer Agent
    ├── delegates to → Critic Agent
    └── aggregates results
         ↓
    Final Response
```

## Project Structure

```
backend/agent-runtime/multi-agent/
├── supervisor/
│   ├── supervisor_agent.py     — Coordinator / router
│   └── task_router.py          — Routes sub-tasks to specialists
├── specialists/
│   ├── researcher_agent.py
│   ├── writer_agent.py
│   └── critic_agent.py
├── shared/
│   ├── tool_registry.py        — Shared ToolRegistry
│   ├── agent_context.py        — Shared task state / message bus
│   └── agent_result.py         — Typed AgentResult
├── protocols/
│   └── agent_protocol.py       — Agent Protocol / ABC
└── tests/
    ├── test_supervisor.py
    └── test_specialist_agents.py
```

## Implementation Patterns

### Agent Protocol

```python
from typing import Protocol
from shared.agent_context import AgentContext
from shared.agent_result import AgentResult

class Agent(Protocol):
    name: str

    async def run(self, task: str, context: AgentContext) -> AgentResult: ...
```

### Typed Agent Result

```python
from dataclasses import dataclass
from enum import Enum

class AgentStatus(Enum):
    SUCCESS = "success"
    FAILURE = "failure"
    STEP_LIMIT_REACHED = "step_limit_reached"
    CANCELLED = "cancelled"

@dataclass
class AgentResult:
    agent_name: str
    status: AgentStatus
    output: str
    steps_taken: int
    error: str | None = None
```

### Supervisor Agent

```python
from protocols.agent_protocol import Agent
from shared.agent_context import AgentContext
from shared.agent_result import AgentResult, AgentStatus
from task_router import TaskRouter

class SupervisorAgent:
    def __init__(self, specialists: dict[str, Agent], router: TaskRouter, max_rounds: int = 5):
        self._specialists = specialists
        self._router = router
        self._max_rounds = max_rounds

    async def run(self, task: str, context: AgentContext) -> AgentResult:
        for round_num in range(self._max_rounds):
            specialist_name = await self._router.route(task, context)
            if specialist_name is None:
                break
            specialist = self._specialists.get(specialist_name)
            if specialist is None:
                break
            result = await specialist.run(task, context)
            context.add_result(specialist_name, result)
            if result.status != AgentStatus.SUCCESS:
                return result
        return context.aggregate_results()
```

### Shared Tool Registry

```python
from dataclasses import dataclass
from typing import Callable, Awaitable

@dataclass
class ToolDefinition:
    name: str
    description: str
    execute: Callable[..., Awaitable[str]]

class ToolRegistry:
    def __init__(self):
        self._tools: dict[str, ToolDefinition] = {}

    def register(self, tool: ToolDefinition) -> None:
        self._tools[tool.name] = tool

    def get(self, name: str) -> ToolDefinition | None:
        return self._tools.get(name)

    def list_tools(self) -> list[ToolDefinition]:
        return list(self._tools.values())
```

## Safeguards

| Risk | Safeguard |
|---|---|
| Infinite delegation loops | Supervisor tracks round count; terminates at `max_rounds` |
| Runaway specialist | Each specialist has its own `max_steps` |
| Slow tool execution | All tool calls have individual timeouts |
| Cascading failures | Supervisor catches `AgentResult.status != SUCCESS`; does not continue blindly |
| Unregistered tool | `ToolRegistry.get()` returns `None`; agent returns structured error |

## Testing Guidance

- Create `FakeAgent` implementations that return deterministic `AgentResult` values
- Test the supervisor with fake specialists — verify delegation sequence and aggregation
- Test the tool registry registration, lookup, and missing-tool handling
- Test cancellation by cancelling the outer task while a fake specialist is mid-execution
- Test step-limit enforcement by providing a fake specialist that never completes

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Specialist calling another specialist directly | All delegation goes through supervisor |
| Supervisor executing tools | Supervisor delegates; specialists use the tool registry |
| Shared mutable state modified concurrently | Use structured context with append-only message lists |
| No step limit on specialists | Each specialist must enforce `max_steps` |
| Swallowing specialist errors | Propagate `AgentResult.status`; log the error |

## Relationship to Steering

This skill governs `backend/agent-runtime/multi-agent/`. The Android agent architecture (`01-architecture.md`, `agent` skill) follows the same principles but is a separate implementation. Security rules apply: validate all tool inputs before execution.
