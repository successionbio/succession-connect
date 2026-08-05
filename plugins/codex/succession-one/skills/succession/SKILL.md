---
name: succession
description: "Route Codex work across Succession ONE surfaces, workspace skills, MCP tools, approvals, artifacts, and browser URLs."
---

# Succession ONE Router

Generated from `docs/surfaces/guides/succession.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `live`
- Category: `router`
- Risk: `medium`

## Surface Metadata

Routes:
- `/`
- `/chats`
- `/today`
- `/inbox`
- `/campaigns`
- `/data`
- `/profile`
- `/library`
- `/settings/mcp`

Primary MCP tools:
- `ping`
- `get_workspace_context`
- `query_today`
- `list_skills`
- `run_skill`
- `search_library`
- `list_artifacts`
- `get_artifact`
- `query_crm`
- `query_inbox`
- `list_approvals`
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
   normally. Context is optional and cannot target another Workspace.
3. If it returns `principal_type=agency`, use Agency-scoped tools without
   context. Before client work, call `manage_managed_workspace` with
   `action="resolve_workspace"`, then pass the exact returned `context`
   object on every Workspace-scoped call, including `get_workspace_context`,
   `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.



<!-- ADR-0072: seven top-level surfaces + Settings footer — Chat, Today, Inbox, Campaigns, Data, Profile, Library, Settings. -->

## Purpose

Use this as the default command set for Succession ONE. It helps Codex decide
which product surface guide, MCP tool, workspace skill, browser URL, approval
queue item, or artifact handoff belongs to the operator's request.

## When To Use

Use this first when the user asks broadly to work in Succession ONE, inspect the
workspace, create or update GTM work, find information, continue a workflow, or
open something live in the app.

## Core Concepts

- Primitive MCP tools inspect or mutate platform state directly.
- Regular Workspace credentials are pinned to one Workspace. Agency credentials
  resolve a managed client through the Agency roster and repeat the returned
  `context` object on every Workspace-scoped call; there is no persistent
  Workspace switch.
- Every dispatched response includes `succession_context`. Confirm its
  Workspace name before relying on a result, especially for Agency work.
- Workspace skills produce bounded, quality-controlled outcomes server-side.
- Browser is the default shared-visibility layer for user-facing workflows.
- Approvals gate live or external actions.
- Artifacts are durable outputs that should be saved and opened for review.
- Product surface data stays inside Succession by default. Do not substitute
  external personal tools, such as Gmail, for a Succession surface unless the
  user explicitly asks for that external account or the Succession tool reports
  a hard unavailable state that you clearly disclose.

## Default Operating Loop

When the operator asks for Succession work, do not require them to name the
surface, tool, URL, trigger type, or verification mode. Infer the likely product
surface from intent, load the relevant guide, and proceed through this loop:

1. Call `ping` without context. Regular users then call
   `get_workspace_context`. Agency users call `get_agency_context`, then
   `manage_managed_workspace` `resolve_workspace` with the exact client UUID,
   slug, or name.
2. Select the relevant surface guide or workspace skill by outcome.
3. Ask concise follow-up questions only for missing information that changes the
   outcome, risk, audience, or irreversible effect.
4. Use safe product defaults when the operator's intent is clear enough.
5. Use MCP for platform state, durable actions, approvals, and artifacts. For
   an Agency-managed client, pass the resolver's `context` unchanged on every
   Workspace-scoped tool call.
6. Automatically open or use Browser for any workflow that creates, updates, or
   verifies something visible in the app.
7. Confirm the user-facing result in Browser when possible, then summarize what
   changed and where to continue.

Do not ask whether to open Browser for app-visible work. Browser is expected for
operator trust and shared confirmation. Skip Browser only for purely
informational read-only tasks or connection failures. If Browser is not
available or not callable in the current Codex thread, say that explicitly and
return the exact Succession ONE route the operator should open. Do not silently
complete an app-visible workflow without either Browser verification or an
explicit Browser-unavailable note.

## Clarifying Questions

Default to one short follow-up question, or a small numbered set, when the
request lacks required context. Ask before guessing when ambiguity affects:

- target object or audience, such as company vs person vs deal;
- destination, such as Maya chat vs artifact vs Today task;
- live external effects, such as send, launch, sync, publish, or an
  Engaged-record write;
- approval policy, schedule, ownership, or workspace-visible behavior;
- substantive content, such as what a "deep dive" should include.

If a safe default is obvious, state it briefly and continue. For example,
"company deep dive action" should default to a single-company CRM action (an
automation with a `crm_action` definition) that opens a new Maya chat with
selected company context, then verify it in Browser.

## Available Actions

- Inspect workspace context, profile readiness, active work, library content,
  Engaged records, approvals, conversations, and agent activity.
- Discover the right workspace skill with `list_skills`.
- Run the selected skill with `run_skill` when the task needs a packaged
  outcome rather than ad hoc tool composition.
- Save final outputs with `manage_artifact` and return app URLs.

## MCP Tools And Actions

- Regular users use `get_workspace_context` before taking scoped Workspace
  action. If it reports `agency_workspace_context_required`, follow the Agency
  guide to resolve a managed Workspace rather than changing credentials.
- Use `query_today` for broad attention, focus, daily queue, and resume-work
  requests.
- Use `list_skills` and `run_skill` for bounded skill outcomes.
- Use `search_library`, `list_artifacts`, and `get_artifact` for source and
  output retrieval.
- Use `query_crm` for company, person, deal, and account context.
- Use `query_inbox` for Succession Inbox queues, lead replies, follow-ups, and
  "what needs my attention in my inbox" requests.
- Use `list_approvals` before assuming a live action has cleared.

## Workspace Skills To Discover

Look for workspace skills by outcome: campaign build, account research,
prospect scoring, message writing, profile audit, signal summary, inbox reply,
or launch preparation. Do not duplicate workspace skill creation in local Codex
skills.

## URLs To Return

Return full absolute Succession ONE app URLs whenever possible so the operator
can open them directly in Codex Browser. Use relative routes only as a last
resort when no app base URL is available.

## Approval Rules

Never send email, launch campaigns, publish to external systems, write Engaged
records, or approve another agent's action without an explicit platform approval
or direct operator instruction that the platform accepts as approval.

## Artifact Rules

Create or update artifacts for drafts, research packs, campaign plans, source
packs, analysis summaries, and reusable documents. Return the artifact URL when
the user should inspect it in the Library.

## Common Workflows

- "Do this in Succession ONE": resolve regular or Agency-managed Workspace
  context, choose the relevant guide, inspect data, ask for missing required
  context, then run tools or a workspace skill.
- "Build something": use source data, run the right skill when available, save
  the result as an artifact, and open Browser or the relevant URL for review.
- "Show me what happened": inspect activity, approvals, artifacts, and
  conversations, then return the most relevant route.
- "What replies need attention?": load workspace context, use the Inbox guide,
  call `query_inbox` with `tab="needs_reply"`, open Browser to
  the full `route` returned by `query_inbox` when available, and summarize the
  Succession Inbox threads with full thread URLs when useful. Do not inspect
  Gmail unless the operator explicitly asks for Gmail.
- "Create an app-visible action or workflow": configure it through MCP, open
  Browser automatically, navigate to the relevant route, exercise the operator
  path, and report whether the visible behavior matches the request. If Browser
  is unavailable, return the route and label the result as platform-configured
  but not visually verified.

## Failure Modes

- Missing token or workspace context: send the user to the full settings/MCP URL
  when known.
- `agency_workspace_context_required`: resolve the requested managed Workspace
  from the Agency roster and repeat its returned context. Do not mint another
  key, edit MCP configuration, or restart the client.
- Tool result is permission-filtered: report the boundary instead of guessing.
- Skill is absent: explain that no matching workspace skill exists and fall
  back to primitive tools only when the task remains safe.
- Browser cannot access signed-in state: return the app URL and continue via MCP.

## Human-Only Exceptions

Humans own final launch, send, publish, irreversible delete, production secret,
billing, and policy decisions unless the platform approval flow explicitly
authorizes the action.
