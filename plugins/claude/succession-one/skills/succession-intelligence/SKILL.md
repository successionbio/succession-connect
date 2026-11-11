---
name: succession-intelligence
description: "Use Data → Discover to inspect Nucleus data, saved company/people lists, list-building filters, tradeshows, buying signals, and evidence that guide GTM work."
---

# Data — Discover

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/intelligence.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `partial`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/data`
- `/data/lists`
- `/data/companies`
- `/data/people`
- `/data/tradeshows`
- `/data/signals`

Primary MCP tools:
- `query_nucleus`
- `manage_intelligence_list`
- `query_events`
- `discover_scigraph`
- `manage_custom_signal_source`
- `run_skill`
- `manage_artifact`
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



<!-- formerly "Intelligence" (own top-level nav item at /intelligence/*), renamed and merged into Data → Discover in ADR-0072. Events are now called Tradeshows; Signals are now called Buying Signals (see signals.md) -->

## Purpose

Data → Discover is the research, evidence, and saved-list layer for companies,
people, tradeshows, topics, buying signals, trials, and source material that
guide GTM decisions. It is the Nucleus-backed, read-mostly band of the Data
surface — distinct from Data → Engaged, which is workspace-owned and
RLS-scoped.

## When To Use

Use this when the user asks for market context, account evidence, scientific or
trial context, buying-signal interpretation, saved company/people lists,
list-building filters, or research that should inform action.

## Core Concepts

- Intelligence should produce evidence-backed recommendations.
- Nucleus data and library artifacts should be combined before making claims.
- Saved Lists are Succession-owned company/people list snapshots or dynamic
  criteria definitions that can feed campaigns, automations, Engaged review,
  and future enrichment.
- Dynamic lists store a criteria definition and materialized rows. Manual
  refresh updates the materialized rows and records a refresh run.
- Data → Discover often feeds Data → Engaged, campaigns, profile, or inbox
  work.

## Available Actions

- Query intelligence records.
- List, open, create, and refresh Saved Lists.
- Build dynamic company lists from Nucleus criteria and dynamic people lists
  from Nucleus company context, Engaged lead criteria, or Inbox reply
  categories.
- Fetch Nucleus company filter metadata before constructing criteria-backed
  searches or dynamic list definitions.
- Discover scientific graph context.
- Configure custom signal sources.
- Run account research or signal summary skills.
- Save evidence packs and briefs as artifacts.

## MCP Tools And Actions

- Use `query_nucleus` for stored intelligence, company filter metadata, company
  searches, and confirmed people discovery. Use `action="company_filter_metadata"`
  before composing Nucleus filters such as company subtype, org type, geography,
  modality, therapeutic area, target, indication, and enrichment status.
- Use `manage_intelligence_list` for Saved Lists:
  `action="list"` with optional `kind`, `source`, `list_membership_type`,
  `refresh_status`, `status`, or `query`; `action="get"` to open rows;
  `action="create_static"` for reviewed rows; `action="create_dynamic"` for
  criteria-backed lists; and `action="refresh_dynamic"` to materialize supported
  dynamic sources. For AI discovery use the explicit provider lifecycle:
  `create_company_provider_list` or approval-gated
  `create_people_provider_list`, then `sync_provider_list` until
  `rows_available=true`; only then use approval-gated `qualify_provider_list`
  with a natural-language `qualification_prompt`.
- For `create_dynamic`, use `dynamic_definition.source`:
  `nucleus_company_criteria` for company query/filter lists,
  `crm_account_criteria` for CRM account/company search lists,
  `nucleus_people_criteria` for people lists tied to a company domain/name,
  `workspace_lead_criteria` for CRM lead-stage/title/email lists, or
  `inbox_reply_criteria` for reply-category lists such as do-not-contact or
  interested.
- Use `query_events` for Tradeshows (Data → Discover → Tradeshows):
  `action="search"` over the cached Nucleus event catalog merged with workspace
  event state (filters: country, city, year, month, source, scope `all|my`,
  status `all|with_people|needs_people`, date range/mode, sort);
  `action="get"` for one event plus its workspace state and linked conference;
  `action="people_list"` to read the saved people list for an event when one
  exists. When `cache_state` is `empty`, or people need to be found for an
  event, send the user to `/data/tradeshows` — cache sync and find-people are
  cost-bearing and run from the web app only.
- Use `discover_scigraph` for scientific and trial discovery.
- Use `search_library` for prior evidence packs.
- Use `run_skill` for account research, trial search, or signal analysis.
- Use `manage_artifact` for durable research outputs.

## Workspace Skills To Discover

Look for account research, find trials by indication, signal summary, qualify
company, score prospect list, and messaging-source skills.

## URLs To Return

Return `/data/lists` for Saved Lists, `/data/companies` for company search/list
review, `/data/people` for people search/list review, `/data/signals` for
Buying Signals review, and `/data` for the overview. Return Data → Engaged,
Campaigns, Profile, or Library URLs for downstream work.

## Approval Rules

Research and saved-list reads are safe. Creating or refreshing Succession-owned
saved lists is an internal workspace write and should be attributed through MCP.
People full-list creation and Company/People qualification require explicit
user confirmation and an approved approval id. Enrichment, external CRM
write-back, campaign enrollment, and external messages require human review
and the relevant approval.

## Artifact Rules

Save research briefs, source packs, evidence summaries, and prioritization
matrices as artifacts, especially when they feed campaigns or Engaged notes.

## Common Workflows

- Build account brief: query Data → Discover, search library, run account
  research if available, save the brief, and return Data plus artifact URLs.
- Build an AI company list: preview with `query_nucleus`, create the full list
  with `create_company_provider_list`, sync until rows are available, then ask
  whether to qualify it. Never treat preview rows as the saved list.
- Build an AI people list: preview with `query_nucleus`, obtain approval for
  `create_people_provider_list`, sync until rows are available, then obtain a
  separate approval before `qualify_provider_list`. Return
  `/data/people?list_id=...`.
- Build a DNC/interested reply list: create a dynamic people list with
  `dynamic_definition.source="inbox_reply_criteria"` and
  `reply_categories`, then refresh it before returning the list URL.
- Explain buying-signal relevance: gather evidence, compare to profile and ICP,
  and return recommended actions with confidence.

## Failure Modes

- Evidence is stale, sparse, or not specific to the buyer.
- Source conflicts are unresolved.
- Data → Discover recommends action without Engaged or profile context.
- Dynamic list refresh fails because upstream Nucleus cache/credentials are not
  ready, Nucleus people criteria lacks a company domain/name, or the dynamic
  source is not refreshable through MCP yet.
- Filters are invented rather than pulled from `query_nucleus
  action="company_filter_metadata"`, causing empty or misleading lists.

## Human-Only Exceptions

Humans approve external claims, legal or regulatory interpretations, and any
campaign or Engaged-record action derived from uncertain evidence.
