# Nexus AI — Figma UI Screen to JIRA Ticket Mapping

## Purpose

Maps every screen and UI section defined in the Nexus AI Figma Master Prompt to its
corresponding Android JIRA delivery ticket. Identifies gaps where the Figma design introduces
UI requirements not originally present in the ticket's Acceptance Criteria.

---

## Complete Screen Map

| Figma Section | Figma Screen / Component | JIRA Ticket | Gap Before Update |
|---|---|---|---|
| **Design System** | Color tokens (dark + light), typography, shapes, spacing, glassmorphism cards, floating bottom nav, microinteractions, skeleton loading, reduced-motion | JIRA-02 | Missing: exact color tokens, glassmorphism spec, floating bottom nav, microinteraction ACs |
| **Onboarding & Auth** | Onboarding flow, sign-in, sign-up, account screen | JIRA-03 | Missing: onboarding screens, account/profile UI ACs |
| **Home** | Hero card, quick action grid, activity section, AI usage card | JIRA-01 + JIRA-02 | Home screen structure is implied; no explicit screen ACs in JIRA-01 |
| **Chat** | Message bubbles, streaming animation, thinking indicator, tool execution card, source card, code block, composer, attachment, voice shortcut | JIRA-05 | Missing: bubble design, streaming indicator, tool/source cards, composer details |
| **Voice** | Immersive orb screen, waveform, live transcription card, 6 interaction states, controls | JIRA-06 | Missing: orb visual, waveform, transcription card, all 6 states |
| **Code Assistant** | Prompt input, language selector, syntax-highlighted code editor, Explain/Generate/Debug/Refactor/Test actions, copy/share | JIRA-07 | Missing: mobile code editor, syntax highlighting, all 5 action modes |
| **Image Generation** | Prompt input, model selection, aspect ratio chips, resolution, advanced settings sheet, progress animation, gallery | JIRA-08 | Missing: aspect ratio chips, gallery, progress animation, advanced settings sheet |
| **Documents** | Document library cards (icon/title/type/size/status), FAB upload, file picker, camera scan, upload progress | JIRA-09 | Missing: document card spec, FAB, camera scan, upload progress bar |
| **RAG Playground** | Question input, answer display, retrieved sources, relevance indicators, expandable passages, pipeline stepper | JIRA-10 | Missing: RAG playground screen, source cards, relevance indicators, pipeline stepper |
| **Model Hub** | Provider cards (OpenAI/Gemini/Ollama/HuggingFace/On-device), model bottom sheet, capabilities, context length, test connection | JIRA-15 | Missing: Model Hub screen, provider card spec, model bottom sheet, hosted/on-device badges |
| **Prompt Studio** | Prompt library, search, system/user prompt, variable chips, model selection, settings, output preview, run, save version | JIRA-04 (Prompt Eng) + NEW | JIRA-04 has no Android UI; needs Prompt Studio screen ACs added |
| **Agent Center** | Agent cards (avatar/name/purpose/model/tools/memory/status), search/filter/create, creation wizard (5 steps) | JIRA-12 | Missing: agent card spec, creation wizard, status badges |
| **Workflow Screen** | Vertical workflow canvas, node cards (Start/LLM/Agent/Tool/Conditional/Memory/End), execution timeline | JIRA-16 (On-Device) → actually JIRA-12 agent + JIRA-15 | New screen — needs ACs in JIRA-12 |
| **Knowledge Hub** | Document library (already mapped to JIRA-09) + RAG playground (JIRA-10) | JIRA-09 + JIRA-10 | See above |
| **MCP & Tool Center** | Two-tab screen: MCP server cards + tool cards, server setup flow | JIRA-11 (MCP) + JIRA-13 (Tools) | Missing: server card spec, tool card spec, setup flow UI ACs |
| **Multi-Agent Center** | Supervisor + specialists, task progress, activity timeline, communication summary | JIRA-12 | Missing: multi-agent UI, agent relationship visualization |
| **Memory Center** | Memory categories (conversation/session/long-term/agent), search, filter chips, memory cards, privacy controls | JIRA-14 | Missing: Memory Center screen, filter chips, retention display, privacy controls |
| **Evaluation & Performance** | Metric cards (requests/latency/tokens/cost/agent success/RAG quality), trend indicators, date filters, model comparison, drill-down | JIRA-18 (Observability) | Missing: performance dashboard screen, metric cards, chart components |
| **Settings** | Account, AI providers, default model, theme, voice, notifications, privacy, memory, integrations, data, about | JIRA-20 | Missing: Settings screen spec, grouped setting cards, switches |
| **Microinteractions** | Orb breathing, streaming text, button press, bottom sheet transitions, progress rings, skeleton loading | JIRA-02 | Missing as explicit ACs |

---

## Gap Summary

### Tickets requiring updates

| JIRA | Gap |
|---|---|
| JIRA-02 | Add: exact design token system, glassmorphism spec, floating bottom nav, microinteraction ACs |
| JIRA-03 | Add: onboarding screens, account/profile UI ACs |
| JIRA-05 | Add: bubble spec, streaming/thinking indicators, tool card, source card, composer spec |
| JIRA-06 | Add: orb visual, waveform animation, transcription card, 6 interaction states |
| JIRA-07 | Add: mobile code editor, syntax highlighting, all 5 action modes as ACs |
| JIRA-08 | Add: aspect ratio chips, history gallery, progress animation, advanced settings bottom sheet |
| JIRA-09 | Add: document card spec, FAB, camera scan, upload progress, Knowledge Hub screen |
| JIRA-10 | Add: RAG playground screen, source cards, relevance indicators, pipeline stepper |
| JIRA-11 | Add: MCP server card spec, tool card spec, two-tab UI, setup flow |
| JIRA-12 | Add: agent card spec, creation wizard, workflow node cards, multi-agent visualization |
| JIRA-14 | Add: Memory Center screen, filter chips, retention display, privacy controls |
| JIRA-15 | Add: Model Hub screen, provider cards, model selection bottom sheet, capability badges |
| JIRA-18 | Add: Evaluation/Performance screen, metric cards, trend indicators, drill-down screens |
| JIRA-20 | Add: Settings screen, grouped setting cards, theme switch, data management |

### Figma screens with no existing ticket gap

| Screen | Reason |
|---|---|
| Home | Covered by JIRA-02 (design system) + JIRA-01 (core). Home is the entry point composed from design system components. |
| Prompt Studio | JIRA-04 covers Prompt Engineering logic; Prompt Studio screen ACs added to JIRA-04 |

---

## Figma Page → JIRA Ticket Map

| Figma Page | Primary JIRA | Supporting JIRA |
|---|---|---|
| Cover | — | — |
| Design System | JIRA-02 | JIRA-01 |
| Onboarding & Authentication | JIRA-03 | JIRA-02 |
| Home | JIRA-02 | JIRA-01 |
| Chat | JIRA-05 | JIRA-02, JIRA-04 |
| Voice | JIRA-06 | JIRA-02 |
| Code Assistant | JIRA-07 | JIRA-02 |
| Image Generation | JIRA-08 | JIRA-02 |
| Models | JIRA-15 | JIRA-04, JIRA-16 |
| Prompt Studio | JIRA-04 | JIRA-02 |
| Agents | JIRA-12 | JIRA-13 |
| Workflows | JIRA-12 | JIRA-15 |
| Knowledge & RAG | JIRA-09 + JIRA-10 | JIRA-02 |
| MCP & Tools | JIRA-11 + JIRA-13 | JIRA-02 |
| Multi-Agent | JIRA-12 | JIRA-14, JIRA-15 |
| Memory | JIRA-14 | JIRA-02 |
| Evaluation | JIRA-18 | JIRA-02 |
| Settings | JIRA-20 | JIRA-02, JIRA-03 |
| Interactive Prototype | All | — |
