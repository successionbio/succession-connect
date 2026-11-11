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
   normally. `client_id` is optional and cannot target another Workspace.
3. If it returns `principal_type=agency` (agency operators only), use
   Agency-scoped tools without `client_id`. Before client work, call
   `manage_managed_workspace` with `action="resolve_workspace"` and
   `params.query` set to the client name or UUID, then pass the returned
   `client_id` on every Workspace-scoped call, including
   `get_workspace_context`, `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.
5. Tools take `action`, `params`, optional `client_id`, and `confirm`.
   Writes return a dry-run plan until they are called again with
   `confirm=true` after the operator approves. Read-only `read_*` aliases
   need no confirmation.


1. Call `list_skills` to see the workspace's effective skill set
   (platform, agency, and workspace skills), already permission-filtered to the
   operator.
2. Pick the skill whose name and description match the operator's intent.
3. Run it with `run_skill({ action: "run", params: { slug, payload } })`. Ask the operator for required
   inputs you do not have.

Skills execute server-side with audit, usage metering, and approval gates. Live
or external actions always require human approval first.
