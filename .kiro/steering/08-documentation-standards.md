# 08 — Documentation Standards

## Core Principle

Documentation explains architecture decisions, module boundaries, and non-obvious design choices. It does not duplicate the source code or restate what the code already makes obvious.

---

## What Must Be Documented

### README.md (Repository Root)
- Project name and purpose
- Core capabilities (brief list)
- Technology stack (brief list)
- Setup instructions: prerequisites, cloning, local.properties configuration, build steps
- How to run the app locally
- How to run tests
- Link to architecture documentation
- Environment configuration guide (what keys are needed, where to put them)

### Architecture Documentation (`docs/architecture/` or in-repo ADRs)
- Clean Architecture layer overview with module map
- AI provider abstraction and how to add a new provider
- MCP architecture and tool registration flow
- Agent execution lifecycle
- RAG pipeline stages and responsibilities
- Memory storage and retrieval design
- Background processing design (what uses WorkManager and why)
- On-device AI integration and fallback policy
- Navigation architecture

### Module-Level Documentation
- Each Gradle module should have a `README.md` or module-level KDoc explaining:
  - What the module is responsible for
  - What it depends on
  - What it exposes to other modules
  - Key public interfaces or entry points

---

## Code Documentation (KDoc)

Write KDoc on:
- All public interfaces and their methods
- Domain entities with non-obvious fields
- Use cases (document the business operation, not the code mechanics)
- Complex algorithms, non-obvious performance decisions, or tricky concurrency patterns
- Anything where a future reader would ask "why does this work this way?"

Do not write KDoc on:
- Obvious getters/setters
- Self-explanatory utility functions
- Private implementation details that are clear from the code

Comments explain **why**, not **what**. The code explains what.

---

## Architecture Decision Records (ADRs)

Record significant architecture decisions as ADRs in `docs/adr/`.

An ADR is required when:
- A major architectural pattern is chosen (e.g., AI provider abstraction strategy)
- A technology is added or replaced
- A significant tradeoff is accepted
- A pattern deviates from the standard architecture rules defined in Steering

ADR format:
```
# ADR-XXX: Title

## Status
Accepted / Superseded by ADR-YYY / Deprecated

## Context
What situation or problem triggered this decision?

## Decision
What was decided?

## Consequences
What are the tradeoffs? What becomes easier? What becomes harder?
```

---

## Specs vs. Documentation

`.kiro/specs/` contains **design and task specifications** for features being built.

`docs/` contains **permanent reference documentation** for the implemented system.

When a feature is implemented and its Jira tickets are done, the relevant Spec should be summarized or referenced in `docs/`. Specs are not substitutes for architecture documentation.

---

## What Documentation Must Not Do

- Duplicate detailed implementation specifications — those belong in `.kiro/specs/`
- Restate Steering rules — Steering files are the authoritative source for rules
- Include secrets, API keys, or credentials
- Reference internal Jira ticket numbers in public-facing docs (architecture docs should be self-contained)
- Become outdated without a plan to update — stale documentation is worse than no documentation

---

## Documentation Maintenance

- Update module README files when a module's responsibilities change significantly
- Update architecture docs when an architectural pattern changes
- Create a new ADR when a previous architectural decision is superseded
- Review documentation accuracy as part of any major refactor PR

---

## Observability Documentation

Document:
- What metrics are collected and why
- What crash reporting is configured
- What logging is enabled in each build variant
- How to interpret key log tags in debug builds

This allows future contributors to understand what the app reports and how to diagnose issues without reverse-engineering the instrumentation.
