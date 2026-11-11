---
description: Explain what you can do with Succession ONE from here.
---

Call `ping` without context first.

- For `principal_type=workspace`, call `list_skills` without context and give
  me a short, grouped overview of the most useful skills.
- For `principal_type=agency` (agency operators only), call
  `get_agency_context` without context and explain the Agency-level actions.
  Ask which managed client I mean before listing client skills; resolve it with
  `manage_managed_workspace` `action="resolve_workspace"` (`params.query` = the
  client name or UUID), then pass the returned `client_id` to `list_skills` and
  every later Workspace-scoped call.

Then explain the two ways to run a skill: its slash command
(`/mcp__succession-one__<slug>`) or just asking you in plain language (you call
`run_skill` with `params.slug` and `params.payload`). Note that writes return a
dry-run plan first, and anything that sends outreach or spends credits
(campaign launch, paid enrichment) requires a human-approved approval — you
prepare it, a human approves it.
