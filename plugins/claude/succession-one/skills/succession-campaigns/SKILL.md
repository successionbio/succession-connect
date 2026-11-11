---
name: succession-campaigns
description: "Use Campaigns to build, inspect, edit, and launch-gate campaigns, their lists, sources, messaging, and launch settings via the conversational Campaign Builder or direct tool calls."
---

# Campaigns

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/campaigns.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `high`

## Surface Metadata

Routes:
- `/campaigns`
- `/campaigns/new`
- `/campaigns/active`
- `/campaigns/drafts`
- `/campaigns/scheduled`
- `/campaigns/archive`

Primary MCP tools:
- `manage_campaign_build`
- `manage_campaign_workbench`
- `manage_campaign_list`
- `manage_campaign_source`
- `manage_custom_signal_source`
- `manage_campaign_launch_settings`
- `manage_campaign`
- `run_skill`
- `search_library`
- `manage_approval`

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



<!-- operator decision 2026-07-09: "Workbench" is retired as user-facing copy. The conversational build surface at /campaigns/new is the Campaign Builder; the record and its detail view at /campaigns/[id] is just "the campaign". Internal tool/table names (manage_campaign_workbench, campaign_workbenches) stay as-is — see CONTEXT.md "Do not use". -->

## Purpose

Campaigns is where profile, Data (Engaged + Discover), sources, lists,
messaging, and connector settings become launch-ready outbound work. The
conversational Campaign Builder at `/campaigns/new` walks the Operator (or an
agent on their behalf) through a fixed requirements checklist; the resulting
record and its tabbed detail view is simply "the campaign" (`/campaigns/[id]`)
once created.

## When To Use

Use this when the user asks to create a campaign, resume an in-flight
campaign build, inspect an existing campaign, build or score a list, draft
messaging, configure launch settings, or prepare launch.

## Core Concepts

- The Campaign Builder assembles campaign state (a Commercial Motion or
  bespoke Profile composition, lists, sources, messaging, launch settings)
  before launch. Never call it a "Workbench" in user-facing copy — only
  internal names (`manage_campaign_workbench`, `campaign_workbenches`) keep
  the old term.
- `manage_campaign_build` is the build's requirements ledger: it tracks which
  of `pace`, `type`, `goal`, `motion`, `offering`, `companies`, `people`,
  `name`, `qualify`, `build_people`, `channels`, `messaging_type`,
  `messaging_personas`, `messaging`, `personalized_review`, `review`, and
  `gate` are satisfied, records answers, and returns the next unsatisfied
  requirement. It does not execute durable writes itself — those still go
  through `manage_campaign_workbench`, `manage_campaign_list`, and
  `manage_approval`.
- Lists, sources, custom signal sources, messaging, and launch settings are
  separate objects tied to one campaign.
- Launch and external sends require approval.
- Campaigns should reuse Profile (including Commercial Motions), Library,
  Engaged, and buying-signal evidence rather than reinventing them.

## Available Actions

- Start or resume a conversational campaign build and read its requirements
  checklist.
- Create, inspect, or update a campaign's state directly.
- Manage lists, sources, custom signal sources, and launch settings.
- Run skills for messaging, prospect scoring, research, and launch preparation.
- Save campaign plans, drafts, and source packs as artifacts.
- Request or inspect launch approvals.

## MCP Tools And Actions

- Use `manage_campaign_build` to read the build's requirements checklist and
  record answers as the Operator (or agent) moves through it — check the
  ledger before assuming a step is unanswered.
- Use `manage_campaign_workbench` for campaign-level state (create/update the
  campaign record itself).
- Use `manage_campaign_list` for target company/people lists bound to a
  campaign (Nucleus-backed; credit-bearing actions such as qualify, build, and
  enrich are approval-gated).
- Use `manage_campaign_source` for source criteria, cadence, and
  review-vs-auto operating policy.
- Use `manage_custom_signal_source` for reusable workspace-level signal
  watches referenced by signal-type campaigns.
- Use `manage_campaign_launch_settings` for send configuration — the only
  section still editable while a campaign is in review state.
- Use `manage_campaign_messaging` to draft, refine, and sample campaign copy
  (`read_campaign_messaging` for `review_status` and coverage). Sample approval
  and manual edits stay human-only in the app.
- Use `manage_campaign` for launch actions: `prepare`, `launch_rehearsal`,
  then `launch_full` (dry-run by default; a live send requires the workspace's
  live-dispatch setting plus an approved approval).
- Use `run_skill` for research, scoring, messaging, and review skills.
- Use `manage_approval` for launch or send gates.

## Workspace Skills To Discover

Look for campaign ideation, content to campaign, write messaging, build
messaging source pack, subject line writer, score prospect list, review
messaging, and review campaign package skills. Launches go through
`manage_campaign`; the older standalone launch skills are retired.

## URLs To Return

Return `/campaigns` for the campaign list, `/campaigns/new` for creation, and
the specific `/campaigns/[id]` URL when a campaign exists. Return approval
URLs for launch decisions.

## Approval Rules

Do not launch, schedule, send, sync to an external sequencer, or publish a
campaign without platform approval. Drafting, scoring, and readiness checks can
run before approval.

## Artifact Rules

Save campaign briefs, message drafts, source packs, scored lists, and readiness
reports as artifacts and attach or reference them from the campaign when
possible.

## Common Workflows

- Build a campaign: call `manage_campaign_build` to see the next unsatisfied
  requirement, gather Profile (including Commercial Motion) and Library
  context, record answers as they're confirmed, build or import a list via
  `manage_campaign_list`, run messaging skills, save artifacts, and return the
  campaign URL once `gate` is satisfied.
- Launch prep: inspect launch settings, list readiness, connector state,
  approvals, and campaign artifacts, then surface blockers.

## Failure Modes

- Missing connector or expired credentials.
- List quality is too low or not consent-safe.
- Messaging lacks source evidence.
- Launch settings are incomplete or unsafe.
- Approval is pending, rejected, or missing.
- Recording a build step out of the requirement order does not skip
  validation downstream; `personalized_review` only applies to 1:1 messaging
  flows (`messaging_type` = personalized) and is auto-satisfied otherwise.

## Human-Only Exceptions

Humans approve launches, live sends, connector pushes, high-volume list imports,
compliance-sensitive claims, and external system writes.
