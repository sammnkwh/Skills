# Claude Code Verification Setup

**The problem:** Claude Code sometimes claims to have completed a task but hasn't actually done so.

**The solution:** A Stop hook and a `/verify` skill that check what Claude claimed to have done and confirm it actually happened.

---

## How it works

- Reads the conversation transcript for context
- Looks at Claude's most recent response to see what it claimed to do
- Skips verification if Claude only responded in chat (no tools used)
- Verifies only state-changing actions — file edits, file creation, bash commands
- Reports back clearly: done or missing

---

## Option 1 — Stop hook (automatic)

Fires after every response. Does nothing unless a `.verify` file exists in the project directory.

**Toggle it:**
```bash
touch .verify   # turn on
rm .verify      # turn off
```

**Hook prompt** (goes in `~/.claude/settings.json`):

```
First, use the Bash tool to check if a file named '.verify' exists in the current working directory (run: test -f .verify && echo exists). If it does not exist, stop immediately and do nothing.

If .verify exists, proceed:

1. Find and read the conversation transcript. The transcript path is available in your context as transcript_path. Read it to understand the full context — what the user asked for, including any back-and-forth discussion that led to the final request.

2. Look at the most recent assistant response in the transcript. Identify specifically what Claude claimed to have done in that response — not the entire original ask, just what was claimed in this response.

3. Check whether Claude used any state-changing tools in that response. State-changing tools are: file edits, file writes, file creation, or bash commands that modify something. Read-only tools (reading files, listing directories, searching, grepping) do not count. If Claude only used read-only tools or no tools at all, skip verification and do nothing.

4. If tools were used, verify that what Claude claimed to have done was actually done: check that files exist and contain the right content, that code changes are present, that commands had the expected effect.

5. Report back clearly in chat. If everything checks out, say so briefly. If something is missing or incorrect, describe specifically what Claude claimed versus what actually exists.

Important: verify only what Claude claimed to have done in this specific response. If Claude paused mid-task to check in with the user, verify only that step — not the entire original request.
```

**Config structure** in `~/.claude/settings.json`:
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "<paste prompt above>",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

---

## Option 2 — `/verify` skill (manual)

Invoke with `/verify` whenever you want to check a specific task.

Create `~/.claude/skills/verify/SKILL.md`:

```markdown
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
```

---

## Let Claude Code set it up for you

Just paste this into Claude Code:

> Set up a global Stop hook and a /verify skill that checks whether Claude actually completed what it claimed to do in each response. The hook should only run when a .verify file exists in the project directory. It should skip verification if Claude only used read-only tools. It should verify only what Claude claimed in the most recent response, not the entire original ask. The skill should be at ~/.claude/skills/verify/SKILL.md and the hook in ~/.claude/settings.json.
