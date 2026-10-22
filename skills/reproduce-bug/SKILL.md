---
name: reproduce-bug
description: Reproduce a reported bug in the running product, isolate its trigger, and collect evidence for an actionable issue. Use when asked to recreate a bug, confirm a regression, or investigate and report broken behavior.
---

# Reproduce a bug

Establish what fails, under which conditions, and how another person can reproduce it. Keep observed behavior, code-supported explanations, and untested hypotheses distinct. A plausible code path is not a confirmed reproduction.

## Start from the report

- Read the repository's `AGENTS.md`, contribution guide, issue templates, and relevant testing instructions. Follow its tooling and reporting conventions.
- Identify the affected surface and running version or commit before choosing a test checkout. Record any mismatch between the installed app, server, and source.
- Search open and closed issues for the symptom. Read close reasons and linked fixes before calling something a duplicate or regression.
- Ask for missing details only when they affect the experiment. Continue independent investigation while waiting. Existing authorization for reproduction or browser use remains valid; do not ask for it again.

For T3 Code, read [the T3 Code adapter](references/t3code.md) before starting a runtime or inspecting its state.

## Build a feedback loop

Use the real product and the smallest fixture that still triggers the reported failure. Reuse a healthy task-owned runtime; otherwise start an isolated instance and retain its terminal session or PID. Keep reproduction artifacts outside tracked source unless they are intended regression tests.

Never write to live user databases or redirect a dev server to production state. Copy only the data needed, through a consistent read-only snapshot or the app's export path. Prefer neutral synthetic data for public evidence. Do not copy credentials, active work, or scheduled jobs into a fixture that could execute them.

Choose a runnable signal for the actual symptom: a real UI interaction, focused test, command, request replay, or small harness. A scrolling failure needs real wheel, touch, or keyboard input as appropriate; setting `scrollTop` alone does not exercise gesture handling. Inspect the UI before acting and verify the resulting state.

Run the trigger again. Record the reproduction rate for intermittent failures. A setup error or failed automation call does not prove the reported product bug.

## Isolate and explain

Once the failure reproduces, list plausible causes and the observations that would distinguish them. Test one variable at a time, retaining a working control. For example, compare an embedded frame with plain text, or a running turn with an idle thread, without changing both at once.

Instrument the boundary that separates those cases. Keep a short trace of relevant events and state transitions, rather than dumping the application. If adding temporary logs, give them a unique marker and remove them afterward. Do not patch the behavior while collecting the baseline.

A root-cause claim needs a distinguishing experiment or an execution trace that supports the mechanism. Source inspection can narrow the investigation; label explanations that have not been verified in the runtime.

If reproduction remains blocked, report the exact failing prerequisite, what was attempted, and the smallest missing input. Do not replace the requested product with a mock and present it as proof.

## Retain useful evidence

Keep the exact version, setup and trigger commands, expected and observed results, and local artifact paths. Capture a short recording for timing or interaction failures. Give any automated assertion a user-visible meaning, such as whether the viewport remains at the reader's chosen message.

Review everything before a public upload. Exclude credentials, pairing URLs, personal conversations, unrelated tabs, account details, and machine-specific private paths. Use a neutral reproduction rather than publishing a live-session capture that contains sensitive data.

Do not claim a fix or make a PR unless that work is requested. When a fix is in scope, retain the failing feedback loop, make the change, and rerun the original scenario plus appropriate focused checks. Document an absent regression-test seam instead of inventing a shallow test.

## Report or file

An investigation request alone does not authorize posting. If the user requests an issue, prepare and publish the report within that scope without another confirmation round. Follow the current issue template and existing label vocabulary; do not create labels or claim maintainer triage authority.

An existing open issue with the same failure usually needs the new verified evidence rather than a duplicate. A closed report needs checking against its fix and the tested version. Explain any narrower trigger or confirmed regression that warrants a separate issue.

Write for a maintainer who has not seen the conversation:

- Lead with the observable failure and concrete trigger.
- Include deterministic steps, expected versus actual behavior, impact, exact version, and environment.
- Include the minimal trace and recording, plus a workaround only if verified.
- Link relevant source at an immutable commit and related issues. State what remains unverified.
- Preserve the repository's required headings, title prefix, disclosures, and labels. When no template exists, use the fields above.

Write multiline bodies to a UTF-8 file and pass `--body-file` to `gh`; preserve real newlines. Re-read the published issue to verify formatting and attachments. Return a clickable issue link and the observed result. If no new issue was appropriate, explain that outcome and link the existing report.

## Leave a usable workspace

Keep the task-owned runtime and fixture available while the user inspects or iterates. A turn ending is not permission to tear down their inspection environment. Stop only processes you started, using retained sessions or captured PIDs. Remove temporary instrumentation; never kill processes by a broad name or path match.

Adapted from T3 Code's `test-t3-app` workflow and fixture guidance. See the [adapter's source links](references/t3code.md#sources) and [MIT notice](THIRD_PARTY_LICENSE_t3code.txt).
