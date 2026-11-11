---
name: launch-linkedin-campaign
description: "Launch an approved LinkedIn campaign from the Succession ONE Campaign Builder via the Succession MCP, with human approval before anything is sent."
---

# Launch LinkedIn campaign

Launch an approved LinkedIn campaign from the Campaign Builder through the
connected Succession MCP server. Use it when the operator wants to put an
approved LinkedIn sequence live; otherwise use `list_skills` (or the
`so-skills` skill) to find a better match.

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


## How To Run

The standalone launch skill is retired. LinkedIn campaigns launch from the
Campaign Builder through `manage_campaign`:

1. Find the campaign: `manage_campaign_build` `action="list"` (or the
   `/campaigns/[id]` URL the operator gives you), then `action="get"`.
2. Check readiness: `read_campaign_messaging` `action="review_status"` and
   `manage_campaign_launch_settings` `action="get"`. If messaging is not
   approved or settings are incomplete, return the campaign URL and stop.
3. Call `manage_campaign` `action="launch_rehearsal"` (a dry run) and show
   the operator the plan.
4. Ask for approval with `manage_approval` `action="create"`. Only after a
   human approves, call `manage_campaign` `action="launch_full"` with
   `confirm=true`.

If you do not have the campaign or the operator's go-ahead, ask first. The
`succession-campaigns` guide has the full flow.

## Approval required

This skill can trigger a live or external action (LinkedIn sending). The platform requires a human-approved approval before anything is sent. Never attempt to bypass that gate.
