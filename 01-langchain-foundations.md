# Chapter 1 — LangChain Foundations

**Goal:** call an LLM from Python with LangChain, understand what a chat model returns,
and connect to a Databricks workspace with the SDK.

## 1. The one line that matters

```python
from langchain_databricks import ChatDatabricks

LLM = ChatDatabricks(endpoint="databricks-gpt-5-4", temperature=0.1, max_tokens=2048)
```

This creates a **chat model** object. LangChain gives every provider the same interface —
`ChatDatabricks`, `ChatOpenAI`, `ChatAnthropic` all work the same way — so code you write
here transfers anywhere.

Two knobs you will use constantly:

| Parameter | What it does | Rule of thumb |
|-----------|--------------|---------------|
| `temperature` | Randomness of the output (0 = deterministic, 1 = creative) | Use **0–0.2** for agents and extraction; higher for brainstorming |
| `max_tokens` | Longest allowed reply | Caps cost and runaway outputs |

> **Why temperature 0.1 in our demo?** The agent parses the LLM's replies with code
> (regexes, JSON parsing). Deterministic output means the parsing doesn't break
> randomly. Whenever an LLM's output feeds a program, keep temperature low.

## 2. `invoke()` — the universal verb

```python
test = LLM.invoke("Say 'ready' and nothing else.")
print(test.content)   # -> ready
```

`invoke()` takes input in, returns a **message object** out. For a string input you get
back an `AIMessage`, and `.content` holds the text. This "smoke test" pattern — a tiny,
cheap call to verify the endpoint works — is worth making a habit before any demo.

Messages come in three roles, and you'll see all of them in our agent:

- `system` — instructions that shape behavior ("You are the OneClick ingestion assistant…")
- `user` (or `human`) — what the person typed
- `assistant` (or `ai`) — what the model replied

## 3. The workspace client

```python
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()
print(w.config.host)   # which workspace am I talking to?
```

`WorkspaceClient()` takes no arguments — it authenticates from your environment
(CLI profile, `~/.databrickscfg`, or `DATABRICKS_HOST` / `DATABRICKS_TOKEN`).
It's your handle to everything in the workspace: `w.connections.list()`,
`w.pipelines.create()`, `w.pipelines.list_pipelines()` — all used in the demo.

> **New-engineer tip:** always print `w.config.host` once at startup. The most common
> "why is nothing here?" bug is being connected to the wrong workspace.

## In our demo

These six lines are the demo's entire foundation. Everything later — intent extraction,
column mapping, metadata classification — is just `LLM.invoke(...)` with smarter prompts.
The SDK calls (`w.connections.list()`, `w.pipelines.create()`) are what make the agent's
answers *real* instead of chat.

## Try it

1. Change the temperature to 1.0 and run the smoke test five times. What changes?
2. Call `LLM.invoke()` with a list of message dicts instead of a string:
   `[{"role": "system", "content": "Reply in pirate speak."}, {"role": "user", "content": "Hello"}]`.
   What does the returned object look like? Print `type(test)` and `test`.
3. Run `w.connections.list()` and print just the connection names. (You'll need Unity
   Catalog connection permissions — if it fails, that's a useful error to read.)

---
← [Course home](README.md) · [Next: Structured output & intent →](02-structured-output-and-intent.md)
