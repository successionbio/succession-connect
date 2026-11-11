---
name: succession-approvals
description: "Use Approvals to inspect, prepare, approve, reject, or explain human-gated Succession ONE actions."
---

# Approvals

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/approvals.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `cross_cutting`
- Risk: `high`

## Surface Metadata

Routes:
- `/today`
- `/agentops`
- `/agency/approvals`
- `/settings/policies`

Primary MCP tools:
- `list_approvals`
- `manage_approval`
- `get_conversation`
- `list_agent_activity`
- `get_workspace_context`

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

Approvals are the cross-surface control point for live or sensitive work. They
preserve human judgment, policy enforcement, and auditability.

## When To Use

Use this when the user asks what is waiting, why something needs approval, to
approve or reject an item, or to inspect an approval target.

## Core Concepts

- Approval state is separate from draft state.
- Approval targets may be campaigns, sends, Engaged-record writes, connector
  actions, publishing, or other agent actions.
- Context should be inspected before asking the user to decide.

## Available Actions

- List pending approvals.
- Inspect approval target context.
- Approve, reject, or update an approval when the user explicitly decides.
- Return the source URL for review.

## MCP Tools And Actions

- Use `list_approvals` for queue state.
- Use `manage_approval` for state transitions.
- Use `list_agent_activity` for target run context.
- Use `get_conversation` when the approval came from a chat.
- Use `get_workspace_context` before scoped conclusions.

## Workspace Skills To Discover

Look for approval triage, launch readiness, compliance review, and campaign or
inbox review skills when the approval needs structured analysis.

## URLs To Return

Return `/today` for the general queue, `/agentops` for run context,
`/agency/approvals` for agency review, and the source surface URL for the target
record.

## Approval Rules

Only change approval state when the user clearly asks for that action and the
platform policy allows it. Summarize what will happen before approving high-risk
items.

## Artifact Rules

Save approval packets, readiness reports, or compliance notes as artifacts when
the decision needs a durable record.

## Common Workflows

- Review approval: list approvals, inspect target, summarize risk and outcome,
  then ask or apply the user's decision.
- Explain blocker: inspect approval state and source run, then return the
  waiting item URL.

## Failure Modes

- Approval target is missing or permission-filtered.
- User asks to approve without enough context.
- Approval was already resolved by another operator.
- Source object changed after approval was created.

## Human-Only Exceptions

Humans own approval decisions for live sends, launches, publishing, Engaged-
record writes, policy changes, and customer-facing commitments.
