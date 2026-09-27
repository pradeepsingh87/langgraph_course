# Chapter 5 — Tools & Human-in-the-Loop

**Goal:** give agents real tools with schemas, and learn to pause a graph for human
approval at the moments that matter.

## 1. From dict-of-functions to real tools

Chapter 3's tools were `{"calculate": calculate}` — a name pointing at a function.
That works, but the LLM has to guess the argument format from the prompt. LangChain
tools carry a **schema**:

```python
from langchain_core.tools import tool

@tool
def average_dog_weight(breed: str) -> str:
    """Look up the average weight of a dog breed."""
    ...
```

The `@tool` decorator builds a JSON schema from the type hints and docstring, so the
model knows *exactly* what arguments to produce. With modern function-calling models
(like the one in chapter 1), the LLM emits structured tool calls natively — no regex
parsing needed. `ToolNode` in LangGraph then executes them.

> **This is the "reliable function calling" breakthrough** from the course intro:
> predictable tool use is what made agents practical.

## 2. Human-in-the-loop: pause the graph

The demo asks for human confirmation at every consequential step:

- *"Is this correct? (yes / no)"* — after intent extraction
- *"Accept these mappings? (yes / no)"* — after column mapping
- *"Create this pipeline? (yes / no)"* — before creating anything real

The pattern is always the same: **propose → pause → approve → proceed**.
In hand-rolled code this is a phase that waits for chat input. In LangGraph it's
native:

```python
graph.compile(interrupt_before=["execute_pipeline"])
# graph pauses; human inspects state, then:
graph.invoke(None, config)   # ...resumes with approval
```

`interrupt_before` / `interrupt_after` freeze the graph at a node boundary, persist
the state, and let a human review or edit it before resuming. The demo's
`confirm_plan` phase is exactly an `interrupt_before=["execute_pipeline"]`.

**Design rule:** interrupt before anything irreversible (creating pipelines, sending
data, spending money) and after anything expensive to redo (long discovery runs).

## 3. Config wizards as nodes

The demo's `configure` phase walks a list of questions — defaults on Enter, numbered
choices, `back` to revisit. As a graph, each question is a node (or one node driven
by `config_idx` in state), and `back` is a conditional edge to the previous question.
The key mechanism: **answers live in state** (`config_answers`), so going back just
means decrementing the index and re-asking — the state already holds everything else.

## In our demo

Count the HITL gates in the notebook: `confirm_intent`, mapping review, metadata
review, `confirm_plan`, `name_conflict`. Five pauses. That isn't over-caution — each
one sits before a step whose output feeds everything downstream. Wrong intent →
wrong connections; wrong mappings → wrong data. The policy overrides
(`_apply_classification_policy`) are the exception that proves the rule: governance
requirements apply *automatically* with no human override allowed.

## Try it

1. Convert one `@tool` function to use a Pydantic schema with descriptions. Feed the
   model a vague request and compare the tool-call arguments before/after.
2. Add `interrupt_before=["act"]` to your chapter-4 graph. Inspect the state at the
   pause — can you *edit* the planned action before resuming?
3. The demo echoes every config answer (`Set X = Y`). Why is that important UX for a
   wizard? What would you lose if answers were silent?

---
← [← Chapter 4](04-langgraph-essentials.md) · [Next: Multi-agent systems →](06-multi-agent-systems.md)
