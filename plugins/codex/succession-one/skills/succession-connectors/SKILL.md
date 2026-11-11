---
name: succession-connectors
description: "Use Connectors to inspect and configure Succession ONE integrations, credentials, availability, and external action readiness."
---

# Connectors

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/connectors.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `high`

## Surface Metadata

Routes:
- `/settings/connectors`
- `/settings/channels`
- `/settings/admin/connectors`

Primary MCP tools:
- `manage_connector`
- `manage_linkedin_accounts`
- `get_workspace_context`
- `ping`
- `run_skill`

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



## Purpose

Connectors manage external system availability and credentials for email,
sequencers, CRM, data sources, and other integrations used by agents and
campaigns.

## When To Use

Use this when the user asks to inspect connector status, configure an
integration, troubleshoot credentials, or understand why an external action is
blocked.

## Core Concepts

- Connector state gates many live actions.
- Tokens and credentials are secrets and should never be exposed in chat or
  artifacts.
- Connector readiness does not replace approval requirements.

## Available Actions

- Inspect connector availability and configuration.
- Create or update connector records when explicitly requested.
- Diagnose missing or expired integrations.
- Route users to settings for credential entry.

## MCP Tools And Actions

- Use `manage_connector` for connector inspection and configuration.
- Use `manage_linkedin_accounts` for white-labeled LinkedIn sender accounts
  (Settings → Channels): `action="list"` for connected accounts,
  `action="status"` for one account's connection state, and
  `action="reconnect"` / `action="resync"` to recover a disconnected or stale
  account through the platform's LinkedIn connection.
- Use `get_workspace_context` before workspace-scoped decisions.
- Use `ping` for basic MCP connectivity checks.
- Use `run_skill` only when a workspace skill handles connector readiness or
  setup analysis.

## Workspace Skills To Discover

Look for setup, connector readiness, launch readiness, email sending, LinkedIn sending,
or CRM sync skills.

## URLs To Return

Return `/settings/connectors` for user connector setup and
`/settings/admin/connectors` for admin connector management when available.

## Approval Rules

Credential changes, external system writes, and connector-enabled launches are
high risk. Do not request or display secrets. Use the platform setup flow.

## Artifact Rules

Do not store secrets in artifacts. It is acceptable to save readiness reports or
non-secret setup checklists.

## Common Workflows

- Diagnose blocked launch: inspect connector status, check campaign launch
  settings, surface missing credentials, and return the settings URL.
- Configure connector: clarify integration, open the settings URL, and only
  mutate non-secret metadata through MCP if the user requested it.

## Failure Modes

- Token expired or missing.
- Connector present but workspace policy blocks action.
- User expects Codex to handle OAuth or secrets directly.
- External API outage or permission issue.

## Human-Only Exceptions

Humans enter secrets, authorize OAuth, approve external writes, and decide which
systems are allowed for workspace operations.
