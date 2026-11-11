---
name: succession-settings
description: "Use Settings to inspect workspace setup, team, connectors, usage, policies, MCP tokens, audit log, automations, and operator preferences."
---

# Settings

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/settings.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/settings`
- `/settings/setup`
- `/settings/team`
- `/settings/connectors`
- `/settings/usage`
- `/settings/policies`
- `/settings/mcp`
- `/settings/audit`
- `/settings/profile`
- `/settings/dnc`
- `/automations`

Primary MCP tools:
- `get_workspace_context`
- `get_workspace_profile`
- `manage_connector`
- `update_operator_preference`
- `update_profile`
- `manage_suppression`
- `ping`

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



<!-- ADR-0072: workspace-knowledge-shaped settings moved into Profile. /settings/workspace-profile -> /profile; /settings/reply-agent -> /profile/reply-agent. Automations moved into Settings as a child. -->

## Purpose

Settings is a footer nav item (not a top-level surface) containing plumbing:
workspace setup, access, connectors, usage, policies, MCP tokens, audit logs,
automations, and personal preferences. Workspace-knowledge-shaped configuration
(workspace profile, reply agent guidance) now lives in Profile, not Settings.

## When To Use

Use this when the user asks about setup, token connection, policies, team
configuration, usage, audit, connector readiness, automations, or personal
preferences.

## Core Concepts

- Settings often controls permissions and guardrails for other surfaces.
- Codex and MCP setup is at `/settings/mcp`. Use the full URL with
  `?provider=codex` when returning a Codex-specific setup handoff.
- Secrets, personal tokens, and OAuth credentials should be handled by the app,
  not pasted into chat.
- Workspace profile configuration lives at `/profile`, not Settings. Reply
  agent guidance lives at `/profile/reply-agent`, not `/settings/reply-agent`.
  Route those requests to Profile, not Settings.

### Do-Not-Contact / suppression

- `/settings/dnc` is the workspace's Do-Not-Contact list — identities
  (people or domains) that must never be contacted again by outbound sends,
  regardless of campaign or channel.
- Adding a suppression entry is a safe, low-risk action — it only narrows who
  can be reached.
- Resolving (un-suppressing) an entry re-opens an identity to future outreach
  and is restricted to user-attended callers — do not resolve a suppression
  on behalf of an unattended/background agent run.

## Available Actions

- Inspect workspace context and profile state.
- Check MCP connectivity.
- Route connector setup or token creation to the correct settings URL.
- Update operator preferences or profile fields when explicitly requested.
- List, add, or resolve Do-Not-Contact / suppression entries.

## MCP Tools And Actions

- Use `get_workspace_context` for workspace state.
- Use `get_workspace_profile` and `update_profile` for profile settings.
- Use `manage_connector` for connector state.
- Use `update_operator_preference` for personal preferences.
- Use `manage_suppression` for the Do-Not-Contact list: `list` to inspect
  current suppression entries (active by default, newest first), `check` to
  test whether one identity is currently suppressed, `add` to suppress one or
  more identities (safe, idempotent, no approval needed), and `resolve` to
  un-suppress an identity — `resolve` requires a user-attended caller and
  should always be confirmed with the operator first, since it reopens
  someone to future outreach.
- Use `ping` for MCP connectivity checks.

## Workspace Skills To Discover

Look for setup, readiness, profile, connector readiness, policy audit, and usage
analysis skills.

## URLs To Return

Return the precise full absolute settings URL: `/settings/mcp?provider=codex`
for Codex setup, `/settings/connectors` for integrations, `/settings/policies`
for guardrails, `/settings/audit` for audit review, `/settings/profile` for
personal settings, `/settings/dnc` for Do-Not-Contact/suppression, and
`/automations` for automations. Route workspace-profile and reply-agent
requests to `/profile` and `/profile/reply-agent` instead.

## Approval Rules

Settings mutations can affect the whole workspace. Confirm broad policy,
connector, team, and usage changes before applying them. Adding a suppression
entry needs no approval; resolving (un-suppressing) one must be confirmed
with the operator first and only performed on a user-attended call.

## Artifact Rules

Do not store secrets in artifacts. Save setup reports, audit summaries, and
policy recommendations only when they are non-secret and useful for review.

## Common Workflows

- Fix Codex access: run `ping`, get workspace context, identify token, OAuth,
  or scope issue, and return the full `/settings/mcp?provider=codex` URL.
- Check readiness: inspect profile, connector, policy, and usage state, then
  summarize missing setup steps.
- Suppress an identity: confirm the identity with the operator, call
  `manage_suppression` `add`, and return `/settings/dnc`.

## Failure Modes

- Token exists but lacks workspace permission.
- Connector or Codex setup requires browser OAuth / token handoff.
- Admin-only settings are hidden from the operator.
- Usage or policy data is permission-filtered.
- Resolving a suppression entry from an unattended/background context is not
  allowed — the caller must be user-attended.

## Human-Only Exceptions

Humans manage secrets, billing, team access, policy changes, OAuth, and broad
workspace configuration decisions.
