---
name: succession-signals
description: "Use Buying Signals (Data → Discover) to inspect buyer, company, topic, tradeshow, and custom signal sources that inform GTM actions."
---

# Buying Signals

Generated from `docs/surfaces/guides/signals.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `partial`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/data/signals`

Primary MCP tools:
- `manage_custom_signal_source`
- `query_nucleus`
- `discover_scigraph`
- `run_skill`
- `search_library`
- `list_agent_activity`

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



<!-- formerly "Signals" (own top-level nav item at /intelligence/signals), renamed to "Buying Signals" and moved under Data → Discover in ADR-0072 -->

## Purpose

Buying Signals capture market, buyer, company, topic, tradeshow, and custom
triggers that can inform account prioritization, messaging, campaigns, and
follow-up.

## When To Use

Use this when the user asks what buying signals exist, which ones matter, how
to add a signal source, or how signals should drive action.

## Core Concepts

- Buying signals are evidence and triggers; they are not campaigns by
  themselves.
- Signal quality depends on source, recency, relevance, and workspace ICP.
- Custom signal sources should be explicit and auditable.

## Available Actions

- Inspect signal context and source data.
- Discover related scientific or market context.
- Create or update custom signal sources.
- Run signal-summary or prioritization skills.
- Save signal summaries as artifacts.

## MCP Tools And Actions

- Use `manage_custom_signal_source` for custom source configuration.
- Use `query_nucleus` for stored intelligence and signal records.
- Use `discover_scigraph` for scientific or trial context.
- Use `run_skill` for signal summaries or account prioritization.
- Use `list_agent_activity` to inspect automated signal runs.

## Workspace Skills To Discover

Look for weekly signal summary, account prioritization, trigger monitoring,
scientific discovery, and campaign targeting skills.

## URLs To Return

Return `/data/signals` for Buying Signals review. Return Data, Campaigns, or
Library URLs when a signal creates downstream work.

## Approval Rules

Reading and summarizing signals is safe. Creating noisy custom sources or using
signals to trigger external action requires operator confirmation and normal
approval gates.

## Artifact Rules

Save signal summaries, evidence packs, and prioritization rationales as
artifacts when they affect campaign or Engaged-record decisions.

## Common Workflows

- Summarize signals: query recent signals, compare to ICP, run a signal summary
  skill if available, and return recommended next actions.
- Add source: define source, scope, refresh cadence, and owner, then create or
  update it with clear downstream risk.

## Failure Modes

- Source is broad enough to generate noise.
- Signal lacks evidence or recency.
- Scientific discovery returns adjacent but not actionable context.
- Signal-to-action mapping is not configured.

## Human-Only Exceptions

Humans approve new high-volume signal sources, external actions triggered by
signals, and claims derived from scientific or regulatory evidence.
