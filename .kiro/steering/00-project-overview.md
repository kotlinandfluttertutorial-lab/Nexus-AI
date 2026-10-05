# 00 — Project Overview

## Project

**Nexus AI**

## Purpose

Production-oriented Android AI Assistant and full-stack agentic AI learning platform.

Nexus AI covers all 20 modules of the Agentic AI Full Stack roadmap: the Android application demonstrates real-world mobile AI architecture while the backend, model-serving, and learning workstreams cover every underlying concept — from GenAI foundations and transformer embeddings through LangChain, LangGraph, multi-agent systems, evaluation, and production deployment.

## Project Workstreams

| Workstream | Scope |
|---|---|
| Android Application | Mobile client (JIRA-01 – JIRA-20) |
| AI Roadmap Modules | Full-stack learning and engineering (NAI-AI-01 – NAI-AI-20) |

## Android Application Capabilities

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

## AI Roadmap Module Coverage

```
01  GenAI Foundations
02  Transformer Architecture & Embeddings
03  Working with LLMs
04  Prompt Engineering
05  Self-Hosted LLMs with Ollama
06  LLM as a Service with Flask
07  LLM Service Testing
08  Building LLM Services with FastAPI
09  Hugging Face & Open-Source LLMs
10  Advanced LLM Features
11  Agentic AI
12  LangChain
13  Memory in AI Agents
14  Retrieval-Augmented Generation
15  Vector Databases
16  LangGraph
17  Model Context Protocol
18  Multi-Agent Systems
19  AI Evaluation
20  Deployment & Production AI Applications
```

## Repository Structure (Target)

```
nexus-ai/
├── android/           — Android mobile application
├── backend/
│   ├── fastapi-service/   — Primary Python AI backend
│   ├── flask-service/     — Flask LLM service (module 06)
│   ├── model-serving/     — Ollama, Hugging Face, provider wrappers
│   ├── agent-runtime/     — LangChain, LangGraph, agents, multi-agent
│   ├── rag/               — Ingestion, chunking, embedding, retrieval
│   ├── mcp/               — MCP clients, servers, tools
│   └── evaluation/        — Datasets, metrics, reports
├── infrastructure/    — Docker, Cloud Run, CI/CD, monitoring
├── learning/          — Per-module notebooks, experiments, demos
│   ├── 01-genai/
│   ├── 02-transformers-embeddings/
│   └── ... (03 – 20)
├── .kiro/             — Steering, skills, specs, settings
└── docs/              — Architecture, API, deployment docs
```

## Android Technology Stack

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

## Backend / AI Technology Stack

| Area | Technology |
|---|---|
| Language | Python 3.11+ |
| API Framework | FastAPI (primary), Flask (module 06) |
| AI Providers | OpenAI, Gemini, Anthropic, Ollama |
| Open-Source Models | Hugging Face Transformers / Pipelines |
| LLM Orchestration | LangChain, LangGraph |
| Embeddings | OpenAI Embeddings, Sentence Transformers |
| Vector Store | ChromaDB |
| MCP | Python MCP SDK |
| Containerization | Docker, Docker Compose |
| Cloud | Google Cloud Run |
| Monitoring | Cloud Logging, structured logs |
| Testing | pytest, httpx, Bruno/Postman |

## Android Architecture Layers

```
Presentation  →  Domain  →  Data
```

Presentation must never directly access:
- Retrofit
- Room
- MCP transport
- AI provider SDKs
- Database implementation details

## Android AI Architecture

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

## Ticket Namespacing

| Namespace | Workstream |
|---|---|
| `JIRA-XX` | Android application features (01–20) |
| `NAI-AI-XX` | AI roadmap modules (01–20) |

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
