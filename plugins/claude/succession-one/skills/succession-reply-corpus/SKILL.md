---
name: succession-reply-corpus
description: "Use Messaging Insights (Profile → Messaging Insights) to analyze past replies, objections, winning language, and reusable response guidance."
---

# Messaging Insights

Generated from `docs/surfaces/guides/reply-corpus.md`. Do not edit this generated skill by hand.
Update the source guide and run `pnpm --filter succession-connect gen`.

- Status: `partial`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/profile/insights`

Primary MCP tools:
- `query_reply_corpus`
- `query_nucleus`
- `search_library`
- `run_skill`
- `manage_artifact`
- `get_workspace_profile`

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
   normally. Context is optional and cannot target another Workspace.
3. If it returns `principal_type=agency`, use Agency-scoped tools without
   context. Before client work, call `manage_managed_workspace` with
   `action="resolve_workspace"`, then pass the exact returned `context`
   object on every Workspace-scoped call, including `get_workspace_context`,
   `list_skills`, and `run_skill`.
4. Check `succession_context.workspace_name` in every result before relying
   on it. There is no persistent Workspace switch.



<!-- formerly "Reply Corpus" (own top-level nav item at /reply-corpus), renamed "Messaging Insights" and moved under Profile in ADR-0072. System names (reply_corpus tables) survive the UI rename. -->

## Purpose

Messaging Insights is the source area for learning from past reply patterns,
objections, sentiment, and language that should inform future inbox and campaign
responses. It is one of Profile's three children (Full Profile, Profile
Improvements, Messaging Insights).

## When To Use

Use this when the user asks what replies are working, what objections recur,
how to improve response language, or how to build a reusable messaging source.

## Core Concepts

- The corpus informs messaging; it should not send messages directly.
- Evidence should come from stored replies, source packs, and workspace profile
  guidance.
- Summaries should preserve examples and caveats.

## Available Actions

- Query reply and signal context.
- Search existing source packs or messaging artifacts.
- Run workspace skills for messaging-source analysis.
- Save guidance as a reusable artifact.

## MCP Tools And Actions

- Use `query_reply_corpus` for read-only attributed-reply evidence:
  `top_rollups` (top-performing patterns by scope — template, component,
  feature_segment, campaign, persona, or company_type — filterable by
  provider and `min_sample_met_only`), `campaign_insight` (the rollup for one
  `provider_campaign_id`, including full evidence), and `learning_candidates`
  (proposed learnings, defaulting to `pending_review`/`approved`/`applied`,
  filterable by status or candidate type). It is strictly read-only — the
  underlying fingerprint/attribution/classification pipeline and its
  review/approval mutations stay in the web review surfaces.
- Use `query_nucleus` for reply and intelligence data.
- Use `search_library` for existing messaging sources.
- Use `get_workspace_profile` for current voice and positioning.
- Use `run_skill` for corpus analysis or messaging source pack creation.
- Use `manage_artifact` to store reusable guidance.

## Workspace Skills To Discover

Look for reply corpus, objection analysis, messaging source pack, voice audit,
and campaign messaging skills.

## URLs To Return

Return `/profile/insights` for the Messaging Insights surface. Return Library
artifact URLs for saved guidance or source packs.

## Approval Rules

Analysis is safe. Any generated response that will be sent must move through the
Inbox or Campaign approval path.

## Artifact Rules

Save source packs, objection matrices, and messaging guidance in the Library so
future skills can reuse them.

## Common Workflows

- Analyze objections: query corpus data, group objections, identify examples,
  and save a guidance artifact.
- Improve replies: combine corpus patterns with workspace profile guidance and
  produce draft language for review.

## Failure Modes

- Corpus data may be sparse or biased toward one campaign.
- Examples can include sensitive customer text.
- Insights can become stale as offers or ICP change.
- A rollup with `min_sample_size_met: false` is statistically thin — treat it
  as directional, not confirmed, unless the caller explicitly requested
  `min_sample_met_only`.

## Human-Only Exceptions

Humans approve sensitive examples, public claims, legal language, and any
message that will be sent externally.
