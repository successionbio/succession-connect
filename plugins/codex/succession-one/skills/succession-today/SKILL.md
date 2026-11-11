---
name: succession-today
description: "Use the Today surface to inspect the operator queue, recent activity, approvals, reminders, and work that needs attention."
---

# Today

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/today.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `low`

## Surface Metadata

Routes:
- `/today`

Primary MCP tools:
- `get_workspace_context`
- `query_today`
- `list_agent_activity`
- `list_approvals`
- `list_artifacts`
- `list_conversations`
- `search_library`

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

Today is the operator's current work queue: approvals, recent agent activity,
important artifacts, inbox reminders, and work that needs a human decision.

## When To Use

Use this when the user asks what needs attention, what happened today, what is
blocked, what is waiting on approval, or where to resume.

## Core Concepts

- Today is an operational summary, not the source system for every object.
- Queue items usually point to artifacts, approvals, conversations, or runs.
- The right next step often belongs in another surface.
- Agency-attention notifications (unread/read/resolved/snoozed) can also
  surface here for agency-home operators, alongside cross-workspace org
  tasks — see the Agency guide's `manage_notification` documentation for the
  full read/ack/resolve/snooze surface.

## Available Actions

- Summarize open approvals and recent agent activity.
- Find recent artifacts or conversations connected to the queue.
- Identify stale work, failed runs, or missing owner decisions.
- Return deep links to the surface where the user can continue.

## MCP Tools And Actions

- Use `query_today` first for broad "what needs attention", "what should I
  focus on", daily operating brief, and resume-work requests. It returns the
  combined queue plus full URLs for the next live handoff.
- Use `list_approvals` for pending approval work.
- Use `list_agent_activity` for recent or failed agent runs.
- Use `list_artifacts` and `search_library` for recent durable outputs.
- Use `list_conversations` when the user wants to resume a thread.

## Workspace Skills To Discover

Look for daily brief, signal summary, inbox triage, campaign readiness, and
pipeline review skills. Run a skill only when the user wants a produced
analysis, not just an inspection.

## URLs To Return

Return the full absolute URL from `query_today.route` for the full queue.
Return approval, artifact, conversation, campaign, inbox, or Data → Engaged
URLs from tool results when a specific item is the next action.

## Approval Rules

Reading the queue is safe. Resolving, approving, rejecting, or escalating a live
item must use the approval tools and must preserve the platform audit trail.

## Artifact Rules

Create a summary artifact only when the user asks for a shareable brief or the
summary is complex enough to preserve.

## Common Workflows

- Daily operating brief: get workspace context, call `query_today`, inspect the
  most relevant returned sections, open Browser to the returned route when the
  user should review the queue live, and summarize the next decisions.
- Resume blocked work: find the failed or waiting item, inspect its linked
  conversation or artifact, and return the correct continuation URL.

## Failure Modes

- Empty queue does not mean nothing exists; check recent activity if the user
  expects ongoing work.
- Missing activity may be a permission boundary or token problem.
- Do not infer launch readiness from Today alone.

## Human-Only Exceptions

Humans decide prioritization, approval outcomes, and whether to resume or cancel
high-risk work.
