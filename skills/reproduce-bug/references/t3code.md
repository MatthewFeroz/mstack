# T3 Code reproduction adapter

Read the target checkout's `AGENTS.md`, `CONTRIBUTING.md`, `.agents/skills/test-t3-app/SKILL.md`, and `.github/ISSUE_TEMPLATE/bug_report.yml`. Their current instructions take precedence over examples here. Use `test-t3-mobile` for native mobile reproduction.

## Runtime and state

- Read the running server version through `t3_environment_read` when available, and identify the corresponding release commit. Distinguish the desktop client version if it differs. Test the reported version before attributing behavior to current `main`.
- Use the checkout's `vp` workflow. Reuse the task's healthy dev server, or run `vp run dev` from an isolated worktree and retain its terminal session. Read actual ports and the pairing URL from the dev-runner output.
- Worktrees normally use their ignored `.t3` state; an explicit `--home-dir` selects another isolated home. Never point a server at the live `~/.t3/userdata` or open that database for writes. Do not set `VITE_HTTP_URL` or `VITE_WS_URL`: dev uses a single origin and Vite's proxies.
- The first worktree start may seed recent real data automatically. Use `vp run dev --no-seed` when creating a neutral public reproduction, then seed only the synthetic records the test needs.
- Respect setup markers before installing dependencies. If setup failed, inspect the specific failure and fix its prerequisite without changing the project's lockfile or package manager to get around it.

## Browser

Use the app-owned `preview_*` tools in T3's Browser panel. Discover their actual schemas, call `preview_status`, and call `preview_open` when no automation-capable tab is attached. Preserve the returned `tabId` on subsequent calls. Inspect with `preview_snapshot` before interacting and prefer its element references.

Use the complete startup pairing URL once. If expired or consumed, mint a fresh dev pairing link using the checkout's documented command. Never publish a pairing token. A bare origin does not pair a new browser.

Respect control ownership. Read a user-controlled tab through snapshots; operate your own tab or an unclaimed one. Retry actionable failures with corrected arguments. If T3's browser tools are absent or explicitly unavailable, report the exact blocker; do not silently replace this repository's prescribed browser system.

When a report does not establish pointer location, compare ordinary text, nested tool output, and inline HTML as relevant. For follow/scroll behavior, distinguish initial positioning, active streaming, layout changes, and an idle settled thread. Record only the combinations that answer the reported failure.

Check input semantics before interpreting results. In the pinned source, `preview_scroll` calls `scrollBy`; it does not deliver a wheel gesture, and scrolling an iframe element does not scroll its document. Use native keyboard input for a keyboard experiment and record any focus setup. A programmatic-scroll result cannot establish that wheel handling works.

## SQLite fixtures

Read the checkout's `.agents/skills/test-t3-app/references/sqlite-fixtures.md` before direct state inspection or fixture writes.

Start the isolated server once to apply migrations, then stop that server before fixture writes. Inspect current table columns and contract schemas; do not assume historical layouts. Use the repository's `apps/server/scripts/t3-sqlite-state.ts` helper when available, with an explicit isolated base directory.

The active database is `userdata/statev2.sqlite`. Editing `state.sqlite` after V2 exists has no effect. Clients read `projection_projects` and `orchestration_v2_projection_*` tables. Match related project, thread, run, message, and turn-item IDs; preserve schema-valid `payload_json`.

Direct projection fixtures can demonstrate client layout and interaction states, but do not establish backend business behavior or a coherent event history. Use app commands/APIs for that. Do not edit auth tables; use documented pairing/auth commands.

T3 also persists thread snapshots in the browser. Direct projection writes leave event sequences unchanged, so a cached snapshot may hide a fixture update. Verify the fresh server response and rendered content; invalidate only the task's isolated browser cache when necessary. Attachment fixtures also need valid IDs and filenames from the current attachment-store contract, beyond a schema-valid tool result.

## Checks and reports

Use focused checks permitted by the checkout. Do not run repository-wide checks unless requested. This workflow does not authorize a T3 Code PR or merge.

The bug-report template currently uses `[Bug]: ` and sections for Before submitting, Area, Steps to reproduce, Expected behavior, Actual behavior, Impact, Version or commit, Environment, Logs or stack traces, Screenshots/recordings/supporting files, and Workaround. Read it afresh rather than treating this summary as a fixed schema. Checkboxes must describe actions actually completed. Apply only labels supported by the current repository.

Public evidence should come from a neutral fixture. For this kind of investigation, a recording plus a minimal state/event trace is stronger than a screenshot of a private conversation.

## Sources

Adaptation basis, pinned for provenance rather than as the current policy:

- [test-t3-app](https://github.com/pingdotgg/t3code/blob/b5a226e49c52ecab4ddc2d678a7012d97e651a6a/.agents/skills/test-t3-app/SKILL.md)
- [SQLite fixture guidance](https://github.com/pingdotgg/t3code/blob/b5a226e49c52ecab4ddc2d678a7012d97e651a6a/.agents/skills/test-t3-app/references/sqlite-fixtures.md)
- [Bug report template](https://github.com/pingdotgg/t3code/blob/b5a226e49c52ecab4ddc2d678a7012d97e651a6a/.github/ISSUE_TEMPLATE/bug_report.yml)
- [Contribution guide](https://github.com/pingdotgg/t3code/blob/b5a226e49c52ecab4ddc2d678a7012d97e651a6a/CONTRIBUTING.md)
