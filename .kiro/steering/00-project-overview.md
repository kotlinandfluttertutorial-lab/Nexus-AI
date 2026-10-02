# 00 — Project Overview

## Project

**Nexus AI**

## Purpose

Production-oriented Android AI Assistant designed as a modular learning and portfolio project.

Nexus AI demonstrates real-world AI application architecture: multi-provider support, agent workflows, RAG pipelines, MCP integration, on-device AI, and secure background processing — all built on clean Android architecture.

## Core Capabilities

```
Chat
Voice
Code Assistant
Image Generation
Document Processing
RAG (Retrieval-Augmented Generation)
MCP (Model Context Protocol)
Tools
Agents
Memory
AI Orchestration
On-device AI
Background Processing
Observability
Testing
CI/CD
```

## Technology Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| UI | Jetpack Compose + Material 3 |
| Async | Coroutines + Flow + StateFlow + SharedFlow |
| Architecture | MVVM + Clean Architecture |
| DI | Hilt |
| Database | Room |
| Preferences | DataStore |
| Networking | Retrofit + OkHttp |
| Serialization | Kotlin Serialization (or Moshi) |
| Background | WorkManager |
| Testing | JUnit + MockK + Compose UI Testing |

## Architecture Layers

```
Presentation  →  Domain  →  Data
```

Presentation must never directly access:
- Retrofit
- Room
- MCP transport
- AI provider SDKs
- Database implementation details

## AI Architecture

```
UI
 ↓
AI Orchestration
 ↓
AI Provider Interface
 ↓
Provider Implementation (Gemini / OpenAI / On-device / Future)
```

Providers must remain replaceable without changes to Orchestration or UI.

## RAG Pipeline

```
Document → Extraction → Normalization → Chunking → Embedding
       → Vector Storage → Retrieval → Context Assembly → AI Generation
```

## MCP Architecture

```
Agent / Orchestration
 ↓
Application Tool
 ↓
MCP Adapter
 ↓
MCP Transport
```

MCP transport must never be coupled to Compose.

MCP handles: server configuration, connection lifecycle, capability discovery, tool discovery, tool invocation, timeout, cancellation, error handling.

## Agent Lifecycle

```
User Task → Task Understanding → Context Collection → Planning
         → Tool Selection → Tool Execution → Observation
         → Next Step → Completion
```

Agents must operate through application-level tool abstractions only.

## Memory

Conversation memory is persisted behind repository abstractions. Presentation never accesses memory storage directly.

## Background Processing

Use WorkManager for work that must survive UI lifecycle changes (embedding jobs, document indexing, sync).

## Development Workflow

Every implementation task must follow this sequence:

1. Inspect repository
2. Read Steering
3. Read applicable Skills
4. Read Jira ticket + Acceptance Criteria
5. Read relevant Specs
6. Inspect existing reusable components
7. Plan
8. Implement
9. Test
10. Build
11. Verify every AC individually

## Responsibility Model

| Artifact | Responsibility |
|---|---|
| Jira ticket | What must be built |
| Acceptance Criteria | What must be true when done |
| Kiro Prompt | How the ticket should be executed |
| Steering | Permanent project-wide rules |
| Skills | Reusable domain expertise |
| Specs | Detailed module/feature design |
| Repository | Actual implementation |
| Tests + AC Verification | Proof that work is complete |

Steering rules take precedence over ad-hoc implementation decisions.
Skills provide reusable guidance but must not override Steering.
