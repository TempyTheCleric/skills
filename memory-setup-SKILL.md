---
name: memory-setup
description: Create a new living memory document to persist context across sessions
category: general
tags: [memory, setup, persistence, context]
status: published
---

## When to Use

When there is no memory document yet, or the user asks to create, initialize, or start
a memory document for the first time. Do not use if a memory document already exists —
use memory-close to update an existing one.

## Procedure

1. Run `manage_skills` with `action: list` filtered to documents, or `manage_documents`
   with `action: list`, to check whether a document titled "memory" already exists.

2. If it exists, stop and use memory-close instead. Tell the user the document already
   exists and offer to update it.

3. If it does not exist, use `create_document` to create a new document titled "memory"
   with the following sections:

   ```
   # Memory

   _Last updated: [date]_

   ## Who I am
   [Identity, role, working context — stable facts that rarely change]

   ## Active projects
   [What is currently in motion, with just enough context to resume]

   ## Established decisions
   [Things that have been decided and should not be relitigated]

   ## Open threads
   [Unresolved questions, loose ends, things to return to]

   ## Preferences and working style
   [How this person likes to collaborate and communicate, what to avoid]

   ## Recent context
   [Warm context from the last session or two — prune aggressively]
   ```

4. If enough context exists in the current conversation, populate each section from it.
   Keep all entries to one or two sentences. Write in plain prose — no bullets, no
   session summaries, no "in this conversation we discussed" framing.

5. Leave placeholder text in any section where there is not enough information to fill
   it accurately. Do not invent or infer beyond what has been established.

6. Confirm with the user what was written before finalizing. They may want to correct
   or add to it.

## Pitfalls

- Do not create a duplicate if the document already exists — check first.
- Do not append a session log. Each entry should reflect current state, not history.
- Do not fill sections with guesses. Placeholders are better than inaccurate content.

## Verification

- Document titled "memory" exists and is openable.
- All six sections are present.
- No section contains invented or hallucinated content.
- User has confirmed the content is accurate.
