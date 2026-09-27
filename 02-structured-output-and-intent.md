# Chapter 2 — Structured Output & Intent Extraction

**Goal:** get reliable, machine-readable data out of an LLM, and use it to understand
what the user wants.

## 1. The problem: LLMs speak prose, code needs data

Our agent needs facts like *"source system = SAP ECC, source type = database"* — but the
LLM answers in sentences. The standard solution is **structured output**: ask the model
to reply in JSON, then parse it.

## 2. The intent extractor

```python
def _extract_intent(self, msg):
    prompt = f"""Extract intent as JSON only:
{{"source_system": "name or null", "source_type": "database|saas|file|streaming|unknown", "tables": [], "context": "brief"}}
Message: {msg}"""
    raw = self._llm(prompt)
    ...
    return json.loads(clean)
```

Three techniques are packed into this small function:

1. **Constrain the vocabulary.** `"source_type"` may only be one of four values or
   `"unknown"`. Fewer choices = fewer surprises. Notice it even includes an explicit
   escape hatch (`unknown`) instead of forcing a wrong guess.
2. **Strip the markdown fences.** Models love wrapping JSON in ` ``` ` despite being
   told not to. The demo handles it explicitly rather than hoping.
3. **Degrade gracefully.** If parsing fails, it doesn't crash — it treats the raw
   message as the source system with type `unknown`, letting the user continue.

> **Takeaway for your own agents:** never let a parse failure be a dead end. A wrong
> guess the user can correct beats an exception every time.

## 3. `_llm()` — the sliding window

```python
def _llm(self, prompt):
    msgs = [{"role": "system", "content": "You are the OneClick ingestion assistant..."}]
    for role, content in self.history[-8:]:
        msgs.append(...)
    msgs.append({"role": "user", "content": prompt})
    return LLM.invoke(msgs).content
```

Every LLM call in the demo goes through this helper. It builds a fresh message list
each time: a short system prompt, the **last 8** conversation turns, then the new
prompt. The `[-8:]` slice is a sliding window — old turns fall off so the prompt
never grows without bound (context windows are finite, and long prompts cost more
and confuse the model).

**LangChain connection:** this hand-rolled helper is exactly what LangChain's
`RunnableWithMessageHistory` and LangGraph's **checkpoints** do for you — manage
conversation memory so each call sees the right context.

## In our demo

Intent extraction is the demo's front door. The raw sentence *"Sync Salesforce contacts
and opportunities"* becomes `{"source_system": "Salesforce", "source_type": "saas",
"tables": ["contacts", "opportunities"]}` — and every downstream phase (connection
filtering, config wizard, metadata) keys off that JSON.

## Try it

1. Write your own extractor for a different domain — e.g. extract
   `{"cuisine", "price_range", "party_size"}` from restaurant requests. What breaks
   when you remove the allowed-values constraint?
2. Deliberately feed `_extract_intent` a confusing message ("just get me the thing
   from the place") and trace what happens downstream with type `unknown`.
3. Change the history window from 8 to 2 and run a long conversation. At what point
   does the agent start "forgetting"?

---
← [← Chapter 1](01-langchain-foundations.md) · [Next: ReAct agent from scratch →](03-react-agent-from-scratch.md)
