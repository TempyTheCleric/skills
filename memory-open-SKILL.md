---
name: memory-open
description: Load the living memory document at the start of a session to restore context
category: general
tags: [memory, session, context, load]
status: published
---

## When to Use

At the start of a new session, or when the user asks to load memory, restore context,
resume where we left off, or read the memory document. Run this before responding to
any substantive request when a session is beginning cold.

## Procedure

1. Use `manage_documents` with `action: list` to locate the document titled "memory".

2. Use `manage_documents` with `action: read` to open and read the full document.

3. Read it completely before responding to anything else. Do not skim.

4. Treat the document as ground truth for established context:
   - Do not ask for information already in the document.
   - Do not contradict what has been established unless the user explicitly revises it
     in this session.
   - Hold open threads in mind — they may become relevant without being mentioned.

5. Acknowledge to the user that memory has been loaded. A single short line is enough —
   do not summarize the document back to them unless asked.

## Pitfalls

- Do not skip this step because the conversation seems self-contained — load memory
  first, then decide what is relevant.
- Do not repeat back the full memory document contents unprompted.
- Do not treat memory as suggestions — it reflects what has been established.

## Verification

- Document was found and read in full.
- Response does not re-ask for context already in the document.
- User is not prompted to re-explain their working context.
