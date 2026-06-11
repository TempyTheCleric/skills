---
name: memory-close
description: Update the living memory document at the end of a session to persist what was established
category: general
tags: [memory, session, update, persistence]
status: published
---

## When to Use

At the end of a session, or when the user asks to save, update, close, or write memory.
Also use when the user says things like "save what we've established," "remember this
for next time," or "update the memory document."

## Procedure

1. Use `manage_documents` with `action: list` to locate the document titled "memory".
   If it does not exist, use memory-setup instead.

2. Use `manage_documents` with `action: read` to read the current document in full.

3. Review the current session and identify what should persist:
   - Decisions made
   - Facts or context established
   - Preferences or working style signals
   - Open threads or unresolved questions worth carrying forward
   - Anything previously recorded that has been superseded or resolved

4. Choose the right update tool based on scope:
   - **Less than ~50% of the document changes** → use `edit_document` with targeted
     find-and-replace. Make one call per changed section. Format:

     ```
     edit_document title:memory
     <<<FIND
     [exact text to replace]
     <<<REPLACE
     [updated text]
     ```

   - **More than ~50% of the document changes** (major restructuring, many sections
     rewritten) → use `update_document` to replace the full content at once.

5. Apply these rules when writing updates:
   - Consolidate: replace superseded content in place — do not append contradictions.
   - Remove anything that no longer applies or has been resolved.
   - Keep entries to one or two sentences each.
   - Plain prose only — no bullets, no "in this session we discussed" framing.
   - The Recent context section should reflect only the last session or two. Prune
     anything older that has not graduated to a permanent section.

6. After all edits are applied, confirm in chat what changed — a brief summary only.
   Do not output the full document.

## Pitfalls

- Do not append new information without checking whether it contradicts or supersedes
  existing content. Consolidate, do not accumulate.
- Do not use `update_document` for small changes — it replaces everything and is harder
  to recover from if something is lost.
- Do not output the full document in chat. A brief change summary is enough.
- The document is a current snapshot, not a session log. It should reflect state, not
  history.

## Verification

- Document has been updated and saved.
- No section contains contradictory or duplicated information.
- Recent context section reflects only recent warm context, not an accumulated log.
- Change summary was confirmed in chat.
