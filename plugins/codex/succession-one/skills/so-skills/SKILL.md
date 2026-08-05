---
name: so-skills
description: Discover and run ANY Succession ONE skill available to your workspace, including workspace- and agency-specific skills not bundled in this pack.
---

# Succession ONE Skills

Use this when the operator asks for a bounded outcome that should be handled by
a Succession workspace skill instead of primitive tool calls.

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


1. Call `list_skills` to see the workspace's effective skill set
   (platform, agency, and workspace skills), already permission-filtered to the
   operator.
2. Pick the skill whose name and description match the operator's intent.
3. Run it with `run_skill({ slug, payload })`. Ask the operator for required
   inputs you do not have.

Skills execute server-side with audit, usage metering, and approval gates. Live
or external actions always require human approval first.
