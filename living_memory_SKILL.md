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

This skill manages a persistent memory document — a plain markdown file that an AI reads
at the start of a session and updates at the end. It approximates built-in memory persistence
without requiring vector stores, RAG pipelines, or external tooling.

The document is the memory store. The prompts below tell the AI how to read it and how to
write to it. The AI does the extraction and consolidation; the human reviews and saves.

---

## How it works

There are three operations:

1. **Setup** — create the initial memory document (empty template or populated from an
   existing conversation)
2. **Session Open** — locate, load, and orient from the document at the start of a session
3. **Session Close** — extract and consolidate at the end, edit the document in place

In Odysseus, the AI locates documents via `manage_skills` procedure query and edits them
directly using `edit_document` or `update_document`. In other environments, the AI outputs
the full updated document for the human to save manually.

---

## Operation 1: Setup

If no memory document exists yet, create one.

### In Odysseus

Run a `manage_skills` procedure query to check whether a document titled "memory" already
exists. If it does, use `edit_document` or `update_document` as appropriate rather than
creating a duplicate.

If the document does not exist, populate it using `update_document` — either from the
empty template in `template.md`, or extracted from the current conversation
(see below). `update_document` is correct here because the document is being written
from scratch.

**If populating from an existing conversation**, extract:
- Who the person is and what they're working on
- Any decisions or conclusions that were reached
- Open threads or unresolved questions
- Preferences or working style signals

Keep entries tight — one to two sentences each. Confirm with the user before writing;
you may have missed things or misread intent.

### In other environments

Output the empty template from `template.md` and tell the user to save it as
`memory.md`. If populating from a conversation, extract the same fields above and present
the populated document for the user to review and save.

---

## Operation 2: Session Open

At the start of a session, load and read the memory document before responding to
anything else.

### In Odysseus

Run a `manage_skills` procedure query to locate the document titled "memory" and open it.
Read it fully before proceeding.

Use this prompt:

```
Run a manage_skills procedure query to find and open the document titled "memory".
Read it fully before responding to anything else.

Treat it as ground truth for established context — who this person is, what we've decided,
where we left off. Do not ask for context already in the document. Do not contradict what's
been established unless the person revises it in this session.

Hold any open threads in mind. They may become relevant without being mentioned directly.
```

### In other environments

Paste the memory document contents at the top of the conversation or inject into the
system prompt, then use this prompt:

```
A memory document is attached. Read it fully before responding.

Treat it as ground truth for established context — who this person is, what we've decided,
where we left off. Do not ask for context already in the document. Do not contradict what's
been established unless the person revises it in this session.

Hold any open threads in mind. They may become relevant without being mentioned directly.
```

---

## Operation 3: Session Close

At the end of a session (or when the user asks), update the memory document.

### In Odysseus

Choose the right tool based on scope:

- **Small changes** (a few entries added, updated, or removed) → use `edit_document` with
  targeted find-and-replace. Preserves everything else in the document.
- **Major restructuring** (document shape has changed, many sections need rewriting) →
  use `update_document` to replace the full content at once.

For `edit_document`, use this format — one call per changed section:

```
edit_document title:memory
<<<FIND
[exact text to replace]
<<<REPLACE
[updated text]
```

Use this prompt:

```
Update the memory document based on this session.

First, run a manage_skills procedure query to open the document titled "memory".

Then apply changes using edit_document for targeted updates, or update_document if the
document needs significant restructuring.

Rules for what to change:
- Extract decisions made, facts established, preferences revealed, and open threads worth
  carrying forward.
- Consolidate: if something has been revised or superseded, replace it in place —
  do not append a contradiction.
- Remove anything that no longer applies or has been resolved.
- Flag anything unresolved that should be revisited.
- Keep entries tight — one to two sentences each.
- Do not summarize the conversation. Only write what should persist.
- Write in plain prose. No bullet summaries, no "in this session we discussed" framing.

After all edits, confirm in chat what changed — a brief summary only, not the full document.
```

### Without filesystem access (Claude.ai, API, manual setups)

Use the same rules above but return the full updated document in chat for the human to
save manually. Review before saving — the AI may drop things or misread what was decided.

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

- **Odysseus (native)**: use `manage_skills` to locate the memory document. `update_document`
  for setup and full rewrites. `edit_document` for targeted session-close updates.
- **Claude.ai Projects**: paste memory document contents into the project instructions field.
  Session close outputs the full document; copy and replace manually.
- **Obsidian + MCP**: inject `memory.md` via the Local REST API once MCP handling is stable.
- **API / custom setups**: include the document in the system prompt or as a user-turn
  prefill. Session close outputs the full document for manual save.

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
