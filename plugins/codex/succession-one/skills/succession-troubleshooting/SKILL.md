---
name: succession-troubleshooting
description: "Diagnose Succession ONE Codex connection, MCP, permission, run, connector, browser, and workflow failures."
---

# Troubleshooting

Generated from `docs/surfaces/guides/troubleshooting.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `live`
- Category: `cross_cutting`
- Risk: `low`

## Surface Metadata

Routes:
- `/settings/mcp`
- `/agentops`
- `/settings/connectors`
- `/settings/audit`
- `/today`

Primary MCP tools:
- `ping`
- `get_workspace_context`
- `get_agency_context`
- `manage_managed_workspace`
- `list_agent_activity`
- `get_conversation`
- `list_approvals`
- `manage_connector`

## Codex Handoff Rules

- Prefer MCP tool-returned full URLs (`url`, `route`, `library_url`,
  `source_url`, `crm_url`) in final answers so users can open them directly
  in Codex Browser.
- Use Browser automatically for app-visible create, update, and verification
  workflows when Browser is available in the Codex thread: after an MCP write,
  open the returned URL to confirm the change rendered, and use it to walk the
  user through UI-only steps this guide marks human-only.
- Never enter credentials, payment details, or approval decisions in Browser
  on the user's behalf; hand those steps back to the user.
- Ask concise follow-up questions when required context is missing; do not make
  the user know internal trigger, route, or table shapes.

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



## Purpose

Troubleshooting gives Codex a disciplined recovery path for connection,
permission, run, connector, browser, and workflow failures.

## When To Use

Use this when a Succession MCP call fails, Codex lacks workspace context, a run
fails, browser access is blocked, a connector is missing, or the user asks why
something did not work.

## Core Concepts

- Diagnose layer by layer: token, workspace, permission, tool, connector,
  approval, run, browser.
- `agency_workspace_context_required` is not a connection failure. Resolve the
  client through `manage_managed_workspace` `resolve_workspace`, then repeat
  its context on each Workspace-scoped call.
- An MCP invalid-arguments/schema error after sending `context` to an
  Agency-scoped or connection-scoped tool means it must be retried without
  context.
- Prefer current platform state over guessing.
- Return the surface where the user can fix the blocking condition.

## Available Actions

- Check MCP connectivity.
- Inspect workspace context and permissions.
- Inspect recent agent activity and approvals.
- Diagnose connector availability.
- Return setup or review URLs.

## MCP Tools And Actions

- Use `ping` without context first for connection checks and principal-type
  discovery.
- For a regular principal, use `get_workspace_context` for Workspace state.
- Use `get_agency_context` and `manage_managed_workspace`
  `resolve_workspace` when the principal is Agency-scoped.
- Use `list_agent_activity` for run failures.
- Use `list_approvals` for blocked live actions.
- Use `manage_connector` for connector readiness.
- Use `get_conversation` for chat-linked failures.

## Workspace Skills To Discover

Look for diagnostic, readiness, connector, approval, or run audit skills if the
workspace has formal troubleshooting workflows.

## URLs To Return

Return `/settings/mcp` for token or MCP setup, `/settings/connectors` for
integration issues, `/agentops` for run failures, `/today` for pending work, and
`/settings/audit` for audit review.

## Approval Rules

Troubleshooting is read-first. Do not fix by approving, launching, sending, or
changing connector settings unless the user explicitly asks and policy allows it.

## Artifact Rules

Save diagnostic reports only for complex or recurring failures. Include
non-secret evidence and next steps.

## Common Workflows

- MCP failure: run `ping`, inspect workspace context if possible, identify token
  or permission issue, and return `/settings/mcp`.
- Agency context failure: resolve the exact roster client, repeat its returned
  context on the failed Workspace tool, and verify the response's
  `succession_context.workspace_name`. Do not change credentials or MCP config.
- Run failure: list activity, inspect linked conversation or approval, summarize
  the failure, and return `/agentops`.
- Connector failure: inspect connector state and return `/settings/connectors`.

## Failure Modes

- Token expired or missing.
- Workspace permission missing.
- Connector credentials missing.
- Approval pending.
- Browser cannot access signed-in state.
- Tool response is permission-filtered.

## Human-Only Exceptions

Humans handle credentials, OAuth, billing, support escalation, approvals, and
production-impacting fixes.
