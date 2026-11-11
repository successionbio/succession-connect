---
name: succession-agentops
description: "Use AgentOps to inspect agent runs, receipts, costs, failures, approvals, and operational health."
---

# AgentOps

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/agentops.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/agentops`

Primary MCP tools:
- `list_agent_activity`
- `list_approvals`
- `manage_approval`
- `get_conversation`
- `get_workspace_context`

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

AgentOps is the operational control plane for understanding what agents did,
what failed, what cost money, and what requires human review.

## When To Use

Use this when the user asks for run history, debugging, audit, costs, receipts,
failed work, or why an agent did something.

## Core Concepts

- Agent runs should be inspectable by status, trigger, surface, receipts, cost,
  and linked artifacts or approvals.
- AgentOps explains behavior; it is not the primary authoring surface.
- Audit context matters before rerunning or approving work.

## Available Actions

- List recent runs, failures, receipts, and approvals.
- Connect runs to conversations and artifacts.
- Summarize cost, latency, and failure patterns.
- Identify whether a rerun, fix, or human approval is required.

## MCP Tools And Actions

- Use `list_agent_activity` for runs and receipts.
- Use `list_approvals` and `manage_approval` for approval queue state.
- Use `get_conversation` when a run came from chat.
- Use `get_workspace_context` before cross-workspace conclusions.

## Workspace Skills To Discover

Look for diagnostic, audit, or postmortem skills if the user wants a formal
analysis rather than a raw operational summary.

## URLs To Return

Return `/agentops` for run inspection. Return conversation, artifact, approval,
or source surface URLs when one item needs direct review.

## Approval Rules

Approving or rejecting work from AgentOps must preserve the approval audit trail.
Do not mark failed work as resolved without an operator decision.

## Artifact Rules

Save formal incident summaries, audit reports, or run analyses as artifacts.
Short operational answers can stay in the response.

## Common Workflows

- Explain a failure: list activity, inspect the run, connect linked records, and
  summarize root cause plus next action.
- Review approvals from runs: list approvals, inspect target run context, and
  return the approval URL or request operator decision.

## Failure Modes

- Run logs may be incomplete if the run failed before receipt creation.
- Cost or duration can be null for interrupted runs.
- Missing linked artifacts may mean the agent produced only transient text.

## Human-Only Exceptions

Humans own approval outcomes, policy changes, and operational signoff after
high-risk agent failures.
