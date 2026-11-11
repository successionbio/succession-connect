# Changelog

All notable changes to the Succession ONE plugins. Versions follow semantic
versioning.

## 0.2.0 (2026-11-11)

### Changed
- MCP server address is now `https://mcp.succession.bio/one/mcp` for Claude
  Code and Codex (Codex `oauth_resource` too). The 0.1.0 address no longer
  serves MCP. **Sign in again after updating**: sign-ins made before the move
  are not carried over.
- `/so-connect`, `/so-whoami` and `/so-help` match the current sign-in flow:
  browser sign-in, an approval page that binds one Workspace, and `ping`
  fields (`workspace_name`, `principal_type`, `principal.effective_role`,
  `next_step`, `db_reachable`). Personal keys from Settings → API + MCP are
  documented as the headless alternative.
- Workspace routing in every skill uses the live calling convention: tools
  take `action`, `params`, optional `client_id` and `confirm`; agency operators
  pass the `client_id` returned by `manage_managed_workspace`
  `resolve_workspace` (the old `context` object is gone). Error codes are now
  `client_required` and `conflicting_client_selection`.
- Codex skill wrappers call `run_skill` as
  `{ action: "run", params: { slug, payload } }`.
- Campaign guidance covers `manage_campaign_messaging` and the
  `prepare` → `launch_rehearsal` → `launch_full` launch flow.
- Customer-facing text no longer names sending or work-tracking providers.
- Codex launch skills renamed to `launch-linkedin-campaign` and
  `launch-email-campaign`. Both standalone
  launch skills are retired on the platform, so these wrappers now walk the
  Campaign Builder launch flow with approval.
- `succession-agency` is marked **Agency only**. In a customer Workspace it says
  an agency operator is needed and stops. The roadmap and troubleshooting
  guides skip agency-only tools in customer Workspaces.

### Added
- Codex wrappers for the current platform skills `campaign-ideation`,
  `content-to-campaign`, `five-challenges-writer`, `review-campaign`,
  `review-messaging`, `subject-line-writer` and `workshop-prep`.
- claude.ai / Claude Desktop custom-connector instructions and a migration
  note in the README.
- This changelog.

## 0.1.0 (2026-08-05)

- First public release: Claude Code and Codex plugins with surface-guide skills,
  bootstrap commands, platform skill wrappers (Codex) and browser OAuth.
