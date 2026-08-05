---
name: launch-heyreach-campaign
description: "Run the Succession ONE \"Launch HeyReach campaign\" skill server-side via the Succession MCP with human approval for live actions."
---

# Launch HeyReach campaign

Run the Succession ONE **Launch HeyReach campaign** skill through the connected Succession MCP
server. Use it when the operator's request matches this skill; otherwise use
`list_skills` (or the `so-skills` skill) to find a better match.

## Workspace Routing

1. On the first Succession interaction, call `ping` without context.
2. If it returns `principal_type=workspace`, use Workspace-scoped tools
   normally. Context is optional and cannot target another Workspace.
3. If it returns `principal_type=agency`, use Agency-scoped tools without
   context. Before client work, call `manage_managed_workspace` with
   `action="resolve_workspace"`, then pass the exact returned `context`
   object on every Workspace-scoped call, including `get_workspace_context`,
   `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.


## How To Run

Call the `run_skill` MCP tool:

```
run_skill({ slug: "launch-heyreach-campaign", payload: { /* skill inputs */ } })
```

If you do not have the required inputs, ask the operator first. Do not
re-implement the skill inline. The platform executes it server-side with audit,
usage metering, and workspace permissions, then returns a structured result.

## Approval required

This skill can trigger a live or external action (connector: `heyreach`). The platform requires a human-approved approval before anything is sent. Never attempt to bypass that gate.
