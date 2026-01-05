# Hands-Off Mode

Enable automated workflows without manual permission prompts by setting `CLAUDE_HANDSOFF=true`. This mode auto-approves safe, local operations while maintaining strict safety boundaries for destructive or publish actions.

## Quick Start

```bash
# Enable hands-off mode
export CLAUDE_HANDSOFF=true
export HANDSOFF_MAX_CONTINUATIONS=10  # Optional: set auto-continue limit

# Run full implementation workflow without prompts
/issue-to-impl 42
```

## What Gets Handsoff?


### Permission Requests

It uses `.claude/hooks/permission-request.sh` to aut-approve safe operations.
It is a more powerful solution to `settings.json` as it only supports rigid regex patterns.


### Auto-continuations

It will automatically continue operations that require multiple stops by hooking `UserPromptSubmit`, `Stop`, and `PostToolUse` events.
As we only have two main workflows `/ultra-planner` and `/issue-to-impl`, and its intermediate steps are well defined, so it is trivial to auto-continue them.

`UserPromptSubmit`: When user submits a prompt, the hook checks if the prompt `seesion_id` matches a file `.tmp/claude-hooks/handsoff/<session_id>.json`,
which stores the metadata about how many continuations have been done so far and the max allowed continuations.
On `CLAUDE_HANDSOFF=true`, if not exists, it creates this file with:

```json
{
  "workflow": "<workflow_name>",
  "max_continuations": <from HANDSOFF_MAX_CONTINUATIONS or default 10>,
  "continuations_done": 0,
  "state": "in_progress"
}
```

If exists, it does nothing.

`Stop`: On `Stop` event, it means the current session is asking for a further prompt to continue.
It checks if the `.tmp/claude-hooks/handsoff/<session_id>.json` exists, and if so, reads it.
If `continuations_done < max_continuations`, it increments `continuations_done` by 1, saves the file, and auto-continues the session by invoking
a Haiku API call to continue the session with a simple prompt.

```
claude -m haiku -p < EOF
{
   [some template to explain the strategy of continuing a workflow]
   [append five last messages in this session to provide context]
}
EOF > /tmp/claude-hooks/handsoff/continue-response-<session_id>.json
```

`post-stop.py` is written in Python because it is easier to deal with JSON manipulation and iteration in Python than bash.

`PostToolUse`: On `PostToolUse` event, it checks `.tmp/claude-hooks/handsoff/<session_id>.json` again.
If it is a `/ultra-planner` upon first placeholder issue open, it changes `state` to `awaiting_issue_details` and saves the file.
If it is a `/ultra-planner` in `awaiting_issue_details` state upon second time of editing the issue with details, it changes `state` to `done`.
When it is `done`, `Stop` hook will no longer auto-continue.

If it is a `/issue-to-impl`, upon first time of publishing the PR, it changes `state` to `done`.
When it is `done`, `Stop` hook will no longer auto-continue.


## Related Documentation

- [Claude Code Pre/Post Hooks](https://code.claude.com/docs/en/hooks)
- [Issue to Implementation Workflow](workflows/issue-to-implementation.md)
- [Issue-to-Impl Tutorial](tutorial/02-issue-to-impl.md)
- [Ultra Planner Workflow](workflows/ultra-planner.md)
