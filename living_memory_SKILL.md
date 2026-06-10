---
name: living-memory
description: >
  Maintain a persistent memory document across AI sessions — simulating the kind of context
  continuity that would otherwise require built-in memory systems. Use this skill whenever
  the user wants to: set up a living memory doc, update memory at the end of a session,
  load memory at the start of a session, create a "persistent context" or "session state"
  file, or asks how to make an AI remember things across conversations. Also trigger when
  the user says things like "save what we've established," "update my memory file," "help
  me set up session memory," or "I want you to remember this for next time." This skill
  covers the full lifecycle: initial setup, session-open loading, and session-close updating.
---

# Living Memory Skill

This skill manages a persistent memory document — a plain markdown file that an AI reads at the start of a session and updates at the end. It approximates built-in memory persistence without requiring vector stores, RAG pipelines, or external tooling.

The document is the memory store. The prompts below tell the AI how to read it and how to write to it. The AI does the extraction and consolidation; the human reviews and saves.

---

## How it works

There are three operations:

1. **Setup** — create the initial memory document (empty template or populated from an
   existing conversation)
2. **Session Open** — load and orient from the document at the start of a session
3. **Session Close** — extract and consolidate at the end, return an updated document

The human saves the updated file between sessions. The AI never stores anything itself.

---

## Operation 1: Setup

If no memory document exists yet, create one.

**If starting from scratch**, output the empty template from `template.md` and tell the user to save it as `memory.md` (or whatever filename suits their setup).

**If populating from an existing conversation**, read the current conversation and extract:
- Who the person is and what they're working on
- Any decisions or conclusions that were reached
- Open threads or unresolved questions
- Preferences or working style signals

Populate the template with what you find. Keep entries tight — one to two sentences each.
Tell the user to review before saving; you may have missed things or misread intent.

---

## Operation 2: Session Open

When a memory document is provided at the start of a session, read it fully before responding to anything else.

Use this prompt (inject into system prompt, or paste at conversation start):

```
A memory document is attached. Read it fully before responding.

Treat it as ground truth for established context — who this person is, what we've decided, where we left off. Do not ask for context already in the document. Do not contradict what's been established unless the person revises it in this session.

Hold any open threads in mind. They may become relevant without being mentioned directly.
```

---

## Operation 3: Session Close

At the end of a session (or when the user asks), update the memory document.

Use this prompt:

```
Update the memory document based on this session.

Rules:
- Extract decisions made, facts established, preferences revealed, and open threads worth carrying forward.
- Consolidate: if something in the document has been revised or superseded, replace it — do not append a contradiction.
- Remove anything that no longer applies or has been resolved.
- Flag anything unresolved that should be revisited.
- Keep entries tight — one to two sentences each. The document should stay scannable.
- Do not summarize the conversation. Only write what should persist.
- Write in plain prose. No bullet summaries, no "in this session we discussed" framing.

Return the full updated document, not a diff.
```

Review the output before saving. The AI may drop things or misread what was decided.

---

## Document size and pruning

The document loads whole into context. Keep it under 600–800 words or it starts crowding
out working space for the actual session.

Prune aggressively. The failure mode is accumulation — the document becomes a log instead
of a snapshot. Ask yourself: does this entry still affect how the AI should behave? If not,
cut it.

The "Recent context" section is meant to expire. Things in Recent context should graduate
to a permanent section or get dropped within a session or two.

---

## Compatibility notes

This skill works in any environment where you can pass a file or text block at session
start:

- **Obsidian + MCP**: inject `memory.md` via the Local REST API or as a system prompt
  attachment in Odysseus/OpenWebUI
- **Claude.ai Projects**: paste memory document contents into the project instructions field
- **API / custom setups**: include the document in the system prompt or as a user-turn
  prefill before the first message
- **Manual**: paste the document at the top of each new conversation

---

## Reference files

- `template.md` — blank memory document template to copy and fill

---

## What this is not

This is not RAG. Nothing is retrieved by query. The whole document loads every time, which
is why size discipline matters.

This is not the same as Claude's built-in memory (Anthropic's summary persistence). That
system runs automatically in the background. This skill is a manual, portable equivalent
you control — works across any AI, any tool, any setup.
