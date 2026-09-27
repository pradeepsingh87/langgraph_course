# LangGraph for New Engineers
### From LangChain basics to multi-agent systems — ending with a real demo

A hands-on course built around the **OneClick Ingestion Agent**: a chat agent that turns
*"Ingest SAP ECC sales data"* into a real, governed Databricks pipeline. You'll learn every
concept by studying the actual notebook code that powers it, then rebuild it the LangGraph way.

## What you'll build

By the end of this course you will understand how the OneClick demo works end-to-end:

1. A user chats: *"Sync Salesforce contacts and opportunities."*
2. The agent extracts intent (source system, source type, tables).
3. It lists real Unity Catalog connections via the Databricks SDK.
4. It discovers tables using Lakehouse Federation (`INFORMATION_SCHEMA` on the source).
5. It maps columns with ADH naming rules and classifies data (PHI / PII / Financial…).
6. Classification-driven policies auto-apply governance (audit logging, masking).
7. A config wizard collects connector options — with `back` and `restart` navigation.
8. A human approves the plan, and the agent creates a real Lakeflow pipeline via the SDK.

## Syllabus

| # | Chapter | What you learn |
|---|---------|----------------|
| 1 | [LangChain foundations](01-langchain-foundations.md) | `ChatDatabricks`, `invoke()`, messages, temperature, `WorkspaceClient` auth |
| 2 | [Structured output & intent](02-structured-output-and-intent.md) | JSON extraction, fence-stripping, graceful degradation, sliding-window history |
| 3 | [ReAct agent from scratch](03-react-agent-from-scratch.md) | The Thought→Action→Observation loop, LLM vs runtime jobs |
| 4 | [LangGraph essentials](04-langgraph-essentials.md) | State, nodes, edges — rebuild the ReAct loop as a graph |
| 5 | [Tools & human-in-the-loop](05-tools-and-human-in-the-loop.md) | Tool calling, approval gates, interrupts, config wizards |
| 6 | [Multi-agent systems](06-multi-agent-systems.md) | Supervisor pattern, specialists, shared state, policy chains |
| 7 | [Capstone: OneClick in LangGraph](07-capstone-oneclick-in-langgraph.md) | Map the 12 demo phases onto a graph; build it yourself |

## Prerequisites

- Python basics (functions, dicts, classes).
- A Databricks workspace you can run notebooks in (the demo uses `langchain-databricks` and the Databricks SDK).
- No prior LangChain or LangGraph experience needed — that's what chapter 1 is for.

## How to use this course

Each chapter follows the same rhythm:

1. **Goal** — what you'll be able to do when you finish.
2. **Concepts** — the ideas, in plain language, with a diagram.
3. **In our demo** — how the concept shows up in the OneClick notebook code you already have.
4. **Try it** — small exercises to make it stick.

> **A note on the demo code:** the notebook you shared is the "before" picture — a hand-rolled
> state machine. Chapters 4–7 teach you to rebuild the same behavior with LangGraph, so you
> see *why* the framework exists: not to do new things, but to make the things you're already
> doing explicit, testable, and persistent.
