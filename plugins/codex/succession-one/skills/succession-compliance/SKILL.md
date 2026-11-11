---
name: succession-compliance
description: "Use Compliance rules to keep Succession ONE actions auditable, permission-aware, approval-gated, and safe for GTM work."
---

# Compliance

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/compliance.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `partial`
- Category: `cross_cutting`
- Risk: `high`

## Surface Metadata

Routes:
- `/settings/policies`
- `/settings/audit`
- `/today`
- `/agentops`

Primary MCP tools:
- `get_workspace_context`
- `list_approvals`
- `manage_approval`
- `list_agent_activity`
- `manage_artifact`

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

Compliance is the cross-cutting rule set for safe Succession ONE operation:
permissions, auditability, approvals, claims, external actions, and data
handling.

## When To Use

Use this when a task involves external action, regulated or sensitive claims,
policy, audit, approval, data export, connector use, or uncertain authority.

## Core Concepts

- Tool access does not equal permission to take high-risk action.
- Approval gates and audit logs are product features, not obstacles.
- Secrets should remain in the app's credential flows.
- Claims should be grounded in evidence and reviewed before use externally.

## Available Actions

- Inspect workspace policy and activity context.
- Review approvals and source runs.
- Save compliance or readiness notes as artifacts.
- Route users to policy or audit settings.

## MCP Tools And Actions

- Use `get_workspace_context` for scope and policy context.
- Use `list_approvals` and `manage_approval` for gated actions.
- Use `list_agent_activity` for audit and run context.
- Use `manage_artifact` for durable review packets.

## Workspace Skills To Discover

Look for compliance review, launch readiness, policy audit, approval triage, and
claim review skills when available.

## URLs To Return

Return `/settings/policies` for guardrails, `/settings/audit` for audit review,
`/today` for approval queue work, and `/agentops` for run context.

## Approval Rules

If an action sends, launches, publishes, writes externally, mutates Engaged
records, changes policy, handles secrets, or creates compliance-sensitive
claims, require an explicit human-approved path.

## Artifact Rules

Save review packets and evidence summaries when they support a decision. Never
store secrets, tokens, private keys, or raw credentials.

## Common Workflows

- Check launch safety: inspect campaign, connector, approval, and evidence
  context, then summarize blockers and return approval or policy URLs.
- Audit action: inspect activity and approvals, connect source artifacts, and
  save a short audit summary if requested.

## Failure Modes

- User assumes Codex can bypass platform policy.
- Evidence is insufficient for a customer-facing claim.
- Scope is unclear across agency and workspace boundaries.
- Secret is pasted into chat.

## Human-Only Exceptions

Humans own legal, compliance, security, policy, launch, send, publish, billing,
and credential decisions.
