---
name: verify
description: Verifies that Claude actually completed what it claimed to do in the most recent response
allowed-tools: Read Bash
---

Review the conversation so far to understand the full context of this session — what the user asked for, including any back-and-forth discussion that led to the most recent request.

Look at the most recent assistant response. Identify specifically what Claude claimed to have done in that response — not the entire original ask, just what was claimed in this response.

Check whether Claude used any state-changing tools in that response. State-changing tools are: file edits, file writes, file creation, or bash commands that modify something. Read-only tools (reading files, listing directories, searching, grepping) do not count. If Claude only used read-only tools or no tools at all, report that there is nothing to verify — Claude only responded in chat.

If state-changing tools were used, verify that what Claude claimed to have done was actually done: check that files exist and contain the right content, that code changes are present, that commands had the expected effect.

Report back clearly in chat. If everything checks out, say so briefly. If something is missing or incorrect, describe specifically what Claude claimed versus what actually exists.

Important: verify only what Claude claimed to have done in the most recent response. If Claude paused mid-task to check in with the user, verify only that step — not the entire original request.
