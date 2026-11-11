---
name: succession-automations
description: "Use Automations to inspect, configure, run, and troubleshoot recurring Succession ONE workflows."
---

# Automations

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/automations.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `high`

## Surface Metadata

Routes:
- `/automations`

Primary MCP tools:
- `manage_automation`
- `list_agent_activity`
- `run_agent`
- `manage_approval`
- `search_library`

## Claude Code Handoff Rules

- Prefer MCP tool-returned full URLs (`url`, `route`, `library_url`,
  `source_url`, `crm_url`) in final answers so users can open them directly.
- When the integrated browser is available in the session, use it for
  app-visible work: after an MCP write, open the returned URL to verify the
  change rendered; open the surface routes above to show results, demo flows,
  or complete UI-only steps this guide marks human-only alongside the user.
- Never enter credentials, payment details, or approval decisions in the
  browser on the user's behalf; hand those steps back to the user.
- Workspace and platform skills surface dynamically as
  `/mcp__succession-one__<slug>` slash commands via MCP prompts — prefer those
  (or `list_skills` -> `run_skill`) over re-implementing a skill inline.
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

Automations define recurring or event-triggered workflows that produce outputs,
create tasks, or request approvals without a user manually starting each run.

## When To Use

Use this when the user asks to create, inspect, edit, pause, run, debug, or
explain recurring Succession ONE workflows.

## Core Concepts

- Automations are durable workspace configuration.
- Runs and outputs should be auditable through agent activity and artifacts.
- Some automation outputs are safe drafts; external actions still need approval.
- Event-triggered automations are not schedules. Prefer the event trigger model
  over legacy schedule labels when the operator asks for CRM actions, inbox
  actions, signal routing, webhook-style behavior, or "when X happens" flows.
- CRM record actions are automations exposed as CRM actions on selected records.
  They should carry selected-record context into their destination.

## Available Actions

- List, inspect, create, update, pause, resume, or run automations.
- Explain trigger, input, destination, and approval behavior.
- Inspect recent runs and failures.
- Save generated plans or outputs as artifacts when useful.

## MCP Tools And Actions

- Use `manage_automation` for automation CRUD and run requests.
- For CRM actions, create or update the automation with the first-class
  `crm_action` and `trigger` fields rather than a manual prompt/schedule shape.
  The canonical shape is `crm_action.object_key = companies|people|deals`,
  `crm_action.label = <operator-facing label>`, `crm_action.selection_mode =
  single|multiple`, `crm_action.destination = maya_new_chat|library_artifact|today_card`,
  and `trigger.kind = event` / `trigger.type = crm.action.clicked` when an
  explicit trigger is supplied.
- Use `list_agent_activity` to inspect automation run history.
- Use `run_agent` only when the operator wants an agent run now and the
  automation model supports it.
- Use `manage_approval` when a run creates a pending approval.

`run_now` is useful for smoke tests only when it can forward input context. When
testing a CRM action, pass selected record metadata if the tool supports it and
verify the generated Maya chat or artifact includes that selected-record
context. If context is dropped, report it as a runner limitation.

## Workspace Skills To Discover

Look for skills that create repeatable operating motions, such as weekly signal
summaries, inbox triage, campaign readiness checks, or CRM hygiene.

## URLs To Return

Return the full absolute automation URL from `manage_automation.url` whenever
available. Return `crm_url` for CRM action verification and Today or approval
URLs when a run is waiting on a decision.

## Approval Rules

Creating or editing an automation that can eventually trigger external action is
high risk. Confirm intent, summarize downstream effects, and rely on platform
approval gates for live sends, launches, or CRM writes.

## Artifact Rules

Save generated automation plans, run summaries, and reusable outputs as
artifacts. Do not store secrets in artifact content.

## Common Workflows

- Build automation: clarify trigger, inputs, cadence, destination, and approval
  policy, then create it and verify the user-facing route in Browser.
- Build CRM action automation: infer CRM plus Automations from the request,
  clarify only missing outcome content, create a `crm.action.clicked` trigger
  with the CRM action definition, open Browser to the CRM route, select a record,
  confirm the action appears, click it when safe, and confirm the destination
  receives selected-record context. If Browser is unavailable, say so and return
  the CRM route; do not describe the action as user-verified.
- Debug automation: inspect recent activity, compare expected trigger behavior,
  and surface the failed run or missing connector.

## Failure Modes

- Connector missing or expired.
- Trigger is too broad and could create noisy work.
- Destination is unsafe without approval.
- User asks for a workflow that belongs as a workspace skill first.
- Legacy schedule/manual shape is used for an event-triggered request.
- CRM action appears in configuration but not on the CRM record Actions menu.
- Selected CRM record context is not forwarded to Maya, artifacts, or run input.
- Automations UI displays a stale schedule label instead of the event trigger
  label. Prefer `automation_triggers` as the source of truth when explaining
  event-triggered automations.

## Human-Only Exceptions

Humans must approve live external actions, policy changes, connector credential
changes, and automations that materially change another user's workflow.
