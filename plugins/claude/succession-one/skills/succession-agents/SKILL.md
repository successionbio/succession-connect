---
name: succession-agents
description: "Use Agents to inspect, configure, and run Succession ONE agent capabilities within workspace policy."
---

# Agents

Maintained in `successionbio/succession-connect` (originally generated from `docs/surfaces/guides/agents.md`).
Tool names checked against the live Succession ONE MCP catalog for v0.2.0.

- Status: `live`
- Category: `surface`
- Risk: `medium`

## Surface Metadata

Routes:
- `/library/agents`

Primary MCP tools:
- `manage_agent`
- `run_agent`
- `list_agent_activity`
- `list_skills`
- `run_skill`
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



<!-- ADR-0072: moved from /agents to /library/agents. Agents has no top-level nav entry — it is one of Library's durable-resource kinds, reachable via Cmd+K or the Library surface. -->

## Purpose

Agents are durable capabilities that can perform work in Succession ONE under
workspace policy, with activity, artifacts, and approvals. Agents has no
top-level nav entry; it is reached through Library or Cmd+K.

## When To Use

Use this when the user asks to inspect available agents, configure one, run one,
compare agent behavior, or understand what an agent can do.

## Core Concepts

- Agents are runtime actors; skills are callable task packages.
- Agent runs should create observable activity and durable outputs.
- External effects remain gated even when an agent initiates them.

## Available Actions

- List, inspect, create, update, or run agents.
- Review recent agent activity and outputs.
- Route bounded jobs to workspace skills when a skill is the better abstraction.
- Escalate approvals created by agent runs.

## MCP Tools And Actions

- Use `manage_agent` for agent configuration.
- Use `run_agent` for direct agent execution when appropriate.
- Use `list_agent_activity` to inspect runs.
- Use `list_skills` and `run_skill` when the task is better expressed as a skill.

## Workspace Skills To Discover

Look for skills that wrap agent-quality outcomes, including account research,
profile audit, message writing, campaign launch readiness, and signal summaries.

## URLs To Return

Return `/library/agents` for agent configuration and discovery. Return
AgentOps, Today, artifact, or approval URLs for run inspection.

## Approval Rules

Agent configuration changes that enable external action are medium to high risk.
Live execution follows the approval policy of the underlying action.

## Artifact Rules

Agents should save reusable outputs to the Library instead of leaving important
work only in run logs.

## Common Workflows

- Run an agent: confirm objective and inputs, start the run, inspect activity,
  and return output or approval URL.
- Configure an agent: inspect current state, make the smallest change, and
  explain policy and approval implications.

## Failure Modes

- Agent exists but lacks a required connector.
- Agent run finishes without an artifact when the user expected a durable output.
- User asks for an agent but a workspace skill is the safer execution path.

## Human-Only Exceptions

Humans own agent policy, external action approvals, and any configuration change
that would allow unattended live execution.
