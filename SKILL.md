---
name: loop-app-code-review
description: Interactive iterative read-only review of an existing application using fresh independent reviewer agents without the orchestrator's conversation history. Does not apply fixes. Collects a finding ledger; later reviewers must not rediscover already recorded issues. Before inspecting anything, ask which severity levels to search, then which review surfaces to search, and wait for explicit answers. Stop by the same loop criteria as loop-code-review-3, scored as no new in-scope findings. Use when the user invokes `/loop-app-code-review`, `$loop-app-code-review`, `/lacr`, `$lacr`, or `lacr`; asks to review an existing program or app without applying patches; or wants a collected backlog of defects. The reviewer must run on exactly the same model and reasoning effort as the main orchestrator; never delegate the review to another model, assistant, agent, or external tool. Track every round in a user-language score table and the status-line state file.
disable-model-invocation: true
---

# Loop App Code Review

## Hard Rule: The Reviewer Runs On The Orchestrator's Own Model

**The reviewer MUST run on exactly the same model and exactly the same reasoning effort as the main orchestrator running this skill.**

**Each scoring pass runs exactly one reviewer.** Never two or more in parallel, and never on different models to compare or combine results.

These are hard requirements, not defaults, preferences, or starting suggestions. This skill never selects, proposes, or falls back to any other model. It never routes the review to another assistant, coding agent, CLI, or review service, no matter what is installed or available in the environment.

If the orchestrator's own model or reasoning effort cannot be established, the loop stops and reports as incomplete. It never proceeds on a substitute.

Full requirements are in **Reviewer Runtime Parity** and **Reviewer Independence**.

## Hard Rule: Collect Findings, Do Not Fix

The entire loop is read-only. Do not edit, stage, commit, reset, stash, push, or otherwise change repository files to address findings. Do not "quickly patch" anything to improve the score.

The deliverable is a finding ledger. Remediation is a later, separate task if the user asks for it.

## Overview

Run an interactive audit loop over an existing application (not only the current task's git diff). First require the user to choose severity levels, then review surfaces. Then use fresh independent read-only reviewer agents on the orchestrator's own model and effort. Merge new in-scope findings into a ledger. Pass the ledger to the next reviewer so it searches for *other* issues, not the ones already recorded. Continue until the scoped acceptance criteria are met.

A score of 9.5/10 means no *new* findings remain in the selected severity levels **and** selected surfaces. It does not mean the application is clean. The backlog may still be long.

## Fast-Path Invocation

If the user explicitly specifies both severity levels and review surfaces in the triggering command (for example `/lacr 1-3 7` or `/lacr levels 1-3 on surfaces 1-4`), normalize both selections, confirm the normalized scope in a single concise sentence, and proceed directly to Step 3 (Workflow) without asking either question. If either axis is missing or ambiguous, ask the missing question(s) sequentially as defined below.

## Mandatory Severity Selection

Before inspecting the repository, running validation, reading code, asking about surfaces, or spawning a reviewer, ask the user which severity levels to search (unless already provided via Fast-Path). This question is mandatory when severity is unspecified. Ask it in the user's language, make it the only substantive response, end the turn, and wait for an explicit answer.

Present all of these options with short plain-language descriptions:

1. **Blockers** — the app cannot be released or meaningfully tested: it does not build or start, the primary flow is completely broken, or a migration can destroy data.
2. **Critical** — use is technically possible, but some users could face catastrophic harm: security or privacy exposure, unauthorized access, money errors, or irreversible data loss.
3. **Serious** — an important or common flow breaks or regularly produces a wrong result: changes do not save, requests duplicate, ordinary users cannot complete the main action, or common input crashes the service.
4. **Medium** — the main flow works, but a plausible edge case or recovery path is meaningfully worse: network recovery, unusual valid input, accessibility, performance, or misleading error behavior.
5. **Minor** — little direct user impact: local maintainability friction, small duplication, weak naming, or low-impact visual and structural imperfections.
6. **Preferences / optional polish** — subjective style, alternative refactors, extra abstraction, or aesthetic preferences that are not defects.
7. **All levels** — search levels 1 through 6.

Ask one direct question such as:

```text
Which levels should this review search? Reply with numbers or names, for example `1-3`, `1,2,4`, or `7`. For an everyday practical review, levels 1-3 are recommended.
```

- Do not silently choose a default.
- Do not start review work until the user answers.
- Accept ranges, lists, names, or an unambiguous natural-language selection.
- If the answer is ambiguous, ask one short clarification and continue waiting.
- Confirm the normalized selected severity in one short sentence, then immediately ask **Mandatory Surface Selection**. Do not inspect the repo between the two questions.

## Mandatory Surface Selection

After severity is answered and before inspecting the repository, running validation, or spawning a reviewer, ask where to search (unless already provided via Fast-Path). This question is mandatory when surfaces are unspecified. Ask it in the user's language, make it the only substantive response, end the turn, and wait for an explicit answer.

These options are *where to look*, not another severity scale:

1. **Primary journey** — the path the application exists to provide. Catches "the main thing does not work". Does not include settings, onboarding, or rare modes unless they *are* that path.
2. **Dependencies of the primary journey** — devices, models, permissions, overlay, hotkeys, single-instance, or equivalent, only when they block the primary path. Not the entire settings UI.
3. **Trust boundaries** — IPC/commands, clipboard, files and model downloads, network, secrets, OS permissions, persistence. Most security and data-loss bugs live here. Not every UI string.
4. **Failures, cancel, races** — stop mid-operation, repeated shortcut, missing device/model, second instance, disconnect. Not a full chaos test of every branch.
5. **Module slice** — one substantive pass per major module, not every file. For dead zones off the primary path. Still not a full tree walk.
6. **Entire codebase** — a deliberate full walk. Expensive; leftover after the 80% pass plus noise. Not the everyday choice.
7. **Practical audit (recommended)** — surfaces **1–4**. Target ~80% of real bugs: primary path, its dependencies, trust boundaries, and typical failures. Not a module-by-module tree walk and not "read every file".

Unless surface 6 is selected (and unless selected severity includes minor/preference for cosmetics), stay out of: i18n strings, pure CSS, generated bindings, vendor, exhaustive test catalogs, and "how I would rewrite the architecture".

Ask one direct question such as:

```text
Where should this review search? Reply with numbers, for example `7`, `1-4`, or `1,3`. For an everyday audit (~80% of real bugs, without reading the whole repository), `7` is recommended. `1-2` is a fast smoke. Add `5` if you suspect dead zones. Choose `6` only for a full walk.
```

- Do not silently choose a default.
- Do not start review work until the user answers.
- Accept ranges, lists, names, or an unambiguous natural-language selection. Treat `7` as surfaces 1–4.
- If the answer is ambiguous, ask one short clarification and continue waiting.
- Confirm the normalized selected surfaces in one short sentence before beginning work.
- Allow the user to change surfaces later. Apply the new set only after explicit confirmation and label subsequent rounds with it.

Do not ask a third question about how deeply to read each file. Depth follows the selected surfaces: inside them, look hard enough to find a concrete selected-severity failure; outside them, do not search except the safety override.

## Severity Scope Contract

Treat the user's selected levels as one axis of the review and acceptance boundary.

- Search deliberately only for selected severity levels.
- Report, score, and continue the loop only because of findings in the selected levels.
- Do not report optional lists of unselected lower-severity findings, reduce the score because of them, or spend tokens polishing them.
- Do not promote or demote a finding merely to fit the selected scope. Classify it by actual impact, likelihood, reach, and recoverability.
- Classify missing or weak tests by the consequence of the regression they fail to protect, not automatically as serious.
- Treat vague complexity, architecture preferences, naming opinions, and speculative hardening as level 6 unless a concrete higher-impact failure scenario is demonstrated.
- If an unmistakable blocker, critical security/privacy exposure, or irreversible data-loss risk is discovered incidentally outside the selected severity or surfaces, surface it once as a safety override. Do not broaden the search.
- Allow the user to change the selected severity later. Apply the new scope only after explicit confirmation and label subsequent rounds with it.

Every score is scoped. A score of 9.5/10 means no *new* actionable findings remain in the selected levels and surfaces; it does not claim unselected levels or surfaces were reviewed or are clean.

## Surface Scope Contract

Treat the user's selected surfaces as the other axis of the boundary.

- Search only on selected surfaces.
- Do not put out-of-surface findings in the ledger or lower the score, except the defined safety override.
- Do not promote cosmetics into a higher severity so they fit a narrow surface.
- Reconstruct just enough of the app map to cover the selected surfaces. Do not use mapping as an excuse to read the whole tree unless surface 6 is selected.

## Review Purpose

Treat review as a structured audit of the existing program within the selected severity and surfaces. Require the reviewer to reconstruct the selected journeys or surfaces, important control or data flow, invariants, and failure behavior. Treat a comprehension obstacle as a finding only when it creates a concrete risk at a selected severity level on a selected surface.

Review comprehensibility alongside correctness, security, privacy, data integrity, UX, and operational behavior. Run focused tests, static checks, builds, or scripts as *evidence* for selected surfaces. Record in-scope red results in the ledger. Never fix them in this skill.

## Reviewer Independence

Independent review means the reviewer runs on the orchestrator's own model and effort and may share the same filesystem, repository state, and applicable project instructions, but must not inherit the parent thread's conversation history, reasoning, assumptions, tool results, or prior review discussion.

Independence comes from a fresh context only. A different model, assistant, or tool is not a source of independence and must never be used to obtain it.

**Exactly one reviewer per scoring pass.** A scoring pass launches a single reviewer agent, waits for its complete output, and produces exactly one score.

- Never run two or more reviewers at the same time, in parallel, or as a fan-out, ensemble, panel, tie-breaker, second opinion, or cross-check.
- Never launch reviewers on different models to compare, combine, average, reconcile, or vote on their findings and scores.
- Never launch an extra reviewer because the first one is slow, cheap to duplicate, or might miss something, or because the environment makes parallel agents easy.
- One round has exactly one reviewer, one model, and one score. If a round would produce more than one score, the design is wrong; stop and run a single reviewer instead.
- Additional reviewers are sequential only: a fresh reviewer starts after the previous round is fully processed, and only for the reasons listed in the workflow.
- Start each scoring reviewer as a fresh agent in an isolated conversation context without parent history.
- Pass a self-contained reviewer prompt containing only the repository location, selected severity levels and definitions, selected surfaces and definitions, the current finding ledger, validation expectations, and evidence the reviewer must independently verify.
- Require the reviewer to inspect the application itself (and git status only as context) before scoring.
- Treat each scoring pass as coming from a fresh reviewer. Follow-up clarification from the same reviewer is not a new scoring pass.

## Reviewer Runtime Parity

The reviewer agent MUST run on exactly the same model and at exactly the same reasoning effort as the orchestrator running this skill. Both are mandatory. Neither may be traded for the other.

Required before every scoring pass:

- Resolve the orchestrator's own running model and its actual reasoning-effort setting first, then launch the reviewer with both identical.
- Match the model exactly: same model, same version or variant identifier. Not a sibling model, not a smaller or larger one from the same family, not a "reviewer" or "reasoning" variant.
- Match reasoning effort exactly: `low` with `low`, `medium` with `medium`, `high` with `high`, `xhigh` with `xhigh`, `max` with `max`, and any other supported level with the identical level.
- Use guaranteed native inheritance when it preserves the orchestrator's exact model and effort while keeping context isolated. Otherwise pass both explicitly at launch.
- Record the exact runtime model name from launch configuration or runtime metadata. Never infer, translate, shorten, or guess it.

Prohibited without exception:

- Selecting, proposing, or defaulting to any model other than the orchestrator's own, for any reason.
- Routing the review to a separate assistant, coding agent, CLI, subscription, API, or external review service, even when one is installed, configured, available, cheaper, faster, idle, or appears better suited to reviewing.
- Treating a different model or tool as a way to achieve reviewer independence. Independence comes only from a fresh, isolated context.
- Promoting the reviewer because it is reviewing, or downgrading its model or effort to save cost, tokens, quota, or time.
- Using a reviewer role, preset, agent type, or configuration whose model or fixed reasoning effort differs from the orchestrator's current model and effort.
- Substituting a stand-in when the orchestrator's model or effort cannot be determined.

When running on platforms with native subagent inheritance (such as `model: inherit` in Antigravity/Gemini or standard subagent tool calls where the host platform automatically preserves the orchestrator's model and reasoning settings), runtime parity is satisfied automatically and does not require explicit textual confirmation of internal reasoning-effort parameters.

If exact model parity or exact effort parity cannot be established (and native inheritance is not available), do not launch the reviewer and do not count a pass. Stop and report the loop as incomplete, naming which parity could not be established.

The only permitted deviation is an explicit, unambiguous instruction from the user to run the reviewer on a specific different model. Never infer this from context, environment, or convenience. When it happens, state the deviation in the round table and in the Final Response.

Always report the model that actually ran.

## Review Scope

Review the existing application on the selected surfaces, not only the current task's git changes.

- Git status, diffs, and branch name are context (dirty tree, unrelated WIP). They are not the review boundary unless the user later narrows to a change set.
- Do not treat unrelated dirty files as out of bounds if they sit on a selected surface; do not expand into them if they do not.
- Preserve the user's worktree. Do not stage, commit, reset, stash, or push.
- If the user named subsystems in the invocation after the two mandatory questions are answered, intersect those names with the selected surfaces; do not replace the surface contract.

### Coverage Breadth and Non-Exclusion Contract

- **Re-examination is allowed and encouraged:** Reviewers are never forbidden from re-examining previously audited files. In complex subsystems, deeper risks, race conditions, or interactions with newly examined modules may only become evident on a second or deeper look.
- **Coverage breadth before acceptance:** A score of 9.5/10 (signaling no new in-scope findings remain) is STRICTLY PROHIBITED until all major components, entry points, and paths comprising the selected surfaces have been visited at least once across the rounds.
- **Audited surface tracking:** Maintain an active list of visited files and subsystems across rounds. In the prompt to subsequent reviewers, pass this list so the reviewer ensures unexamined areas of the selected surface are inspected, while still allowing re-investigation of high-risk components.

## Review Dimensions

Apply these dimensions only where they can produce findings in the selected severity levels on selected surfaces:

- **Comprehensibility:** Reconstruct responsibility, flow, state transitions, invariants, and failure behavior. Raise only a specific obstacle with a concrete selected-level risk.
- **Correctness and operational risk:** Behavioral failures, invalid assumptions, security/privacy exposure, data-integrity problems, poor failure handling, unsafe operational consequences.
- **Test evidence:** Judge whether tests exercise selected-surface behavior. Map coverage gaps to the severity of the behavior left unprotected. Do not turn this into a full test inventory unless surface 6 is selected.
- **Reuse and local fit:** Recommend reuse only when avoiding an existing piece creates selected-level impact.
- **Architecture and conventions:** Finding only when an identifiable project rule or precedent is violated and creates selected-level risk.

Do not chase score-only polish.

## Finding Ledger

Maintain one running ledger for the loop. Each new in-scope finding gets a stable id (`F1`, `F2`, …) that never changes.

Record at least:

- `id`
- selected-level severity
- selected surface(s) it belongs to
- file and line references
- one-line signature (symptom + location)
- plain user impact
- status `open`

**Persistent Project File Output (`audit-ledger.md`):**
- By default, maintain and update the full ledger in a project-local markdown file: `audit-ledger.md` in the project root.
- Do not flood intermediate chat turns with the full ledger text. In intermediate chat updates, output only the compact Score Trajectory Table and a link to `audit-ledger.md`.
- Ensure `audit-ledger.md` contains the full structured details for all findings, grouped by severity.

**System Failure Aggregation:**
- Aggregate homogeneous static analysis, compiler, linter, or test failures into a single composite finding (e.g. `F1: Linter/typecheck errors in X files on surface 3`) rather than cluttering the ledger with dozens of individual tool error entries.

**Next reviewer must not search for recorded issues.** Include the full ledger in the reviewer prompt. Instruct: do not report already recorded items; search for *other* defects in the same selected scope; if the same issue is rediscovered, emit `duplicate-of: ID` with no new id.

The orchestrator merges output:

- Accept new in-scope findings into the ledger and update `audit-ledger.md`.
- Drop duplicates and near-duplicates (same file + same symptom), including paraphrases. Do not rely on the reviewer to be honest about duplicates.
- Do not lower the round score because of duplicates or already-recorded items.
- Continue the loop only because of *new* in-scope findings or a score below 9.5 with a concrete new in-scope issue.

## Workflow

1. Determine scope: use **Fast-Path Invocation** if both severity and surfaces were provided in the command; otherwise complete **Mandatory Severity Selection** and **Mandatory Surface Selection** sequentially.
2. Confirm both scopes in one concise sentence, then inspect just enough to map selected surfaces:
   - Identify entry points and the primary journey if surfaces 1–2 (or 7) are selected.
   - Identify trust-boundary and failure-path code if surfaces 3–4 (or 7) are selected.
   - For surface 5, list major modules and take one slice each.
   - For surface 6, plan a full walk; still score only selected severity.
   - Run `git status --short` only as context. Do not use it as the review boundary.
3. Gather validation evidence for selected surfaces (smallest meaningful tests, typecheck, lint, build, or focused scripts). Record commands and results. If something is red and maps to selected severity on a selected surface, add it to the ledger as an aggregated finding. Do not fix it. If an out-of-scope failure prevents meaningful review, stop and ask whether to expand; do not fix it.
4. Start exactly one independent reviewer, never two or more at once:
   - Apply **Reviewer Runtime Parity** first (native inheritance satisfies parity automatically). If either model or effort cannot be confirmed, stop and report incomplete.
   - Start a fresh isolated reviewer conversation on that same model and effort.
   - Include selected severity, selected surfaces, both contracts, the **current ledger**, and the **list of components/paths audited in prior rounds**.
   - Require read-only independent inspection, a comprehension summary of selected surfaces, new findings with file/line references (or `duplicate-of`), and a scoped numeric score from 1 to 10.
5. Process output:
   - Reject every finding outside selected severity or surfaces except the safety override.
   - Merge new in-scope findings into the ledger and update `audit-ledger.md`. Never apply code fixes.
   - Never accept 9.5+ while the reviewer lists a *new* unresolved in-scope actionable finding, OR while major components of the selected surfaces remain unvisited (Coverage Breadth Contract).
   - If the reviewer scores below 9.5 with no new in-scope actionable findings, ask once what concrete new in-scope issue prevents 9.5. Accept an explicit no-new-in-scope-findings signal when no issue is supplied.
   - If output remains malformed or demonstrates no credible understanding, use a fresh reviewer.
6. Report the completed round:
   - Add it to the running table using **Score Trajectory Report**.
   - In chat, output only the Score Trajectory Table and a clickable link to `audit-ledger.md`.
   - Refresh the status-line score file at the same moment.
7. Repeat:
   - Use a fresh reviewer after merging findings, evidence-based rejection, or a malformed review. Never reuse a score after a new actionable finding was reported.
   - Pass the updated ledger and visited coverage map every time.
   - Accept only when required validation for the selected scope is either green or already in the ledger, all key parts of the selected surfaces have been visited at least once, no *new* in-scope findings remain, and the latest reviewer either scores at least 9.5/10 or explicitly reports no new in-scope actionable findings.
   - Use at most five scoring passes unless the user requests another limit or persistence until acceptance.
   - Treat two unchanged passes that only repeat duplicates, already-recorded items, or out-of-scope comments as stagnation.
   - Pass-limit exhaustion or stagnation without acceptance is incomplete, not success.

## Scoped Scoring Anchors

- **10.0:** No known selected-level defects remain *undiscovered* on selected surfaces in this loop; relevant validation was run; the reviewer found nothing new and the ledger may still list earlier findings.
- **9.5:** No *new* selected-level actionable findings; only unselected or already-recorded items may exist; relevant validation is sufficient.
- **Below 9.5:** At least one *new* meaningful selected-level finding on a selected surface remains unrecorded, or required validation is missing.

The score summarizes only the chosen levels and surfaces. It never overrides concrete new in-scope findings. It never claims the backlog is empty.

## Score Trajectory Report

- Maintain a running scoreboard. After the first round, before each subsequent pass, and once more in the Final Response, print exactly one table in the user's language and plain wording. Do not add an English duplicate.
- For intermediate updates, the table is the entire update with no prose above or below it.
- Use these translated columns:
  - **Round:** completed round number.
  - **Search scope:** selected severity levels and surfaces for that round.
  - **Model:** exact runtime model name that actually reviewed that round; never substitute or guess it.
  - **Score:** scoped X/10 score.
  - **New / ledger:** new in-scope findings this round and total open in the ledger, e.g. `2 / 5`.
  - **Most serious new finding:** blocker, critical, serious, medium, minor, preference, or none in plain words.
  - **Why we continue or stop:** one short plain-language reason.
- Translate findings for a non-programmer and keep every cell to one short line.
- When a score dips, explain that a real new in-scope issue surfaced which earlier rounds missed.

Example in Russian; render it in the user's language with the real runtime model name. `<модель>` below is a placeholder — never print a placeholder or copy an example value into a real table.

| Раунд | Что искали | Модель | Оценка | Новых / в списке | Самая серьёзная новая | Почему продолжаем или стоп |
|-------|-------------|--------|--------|------------------|------------------------|-----------------------------|
| 1 | Серьёзные+; поверхности 1–4 | `<модель>` | 8,0 | 3 / 3 | Серьёзная | Нашли новые замечания — копим список |
| 2 | Серьёзные+; поверхности 1–4 | `<модель>` | 9,5 | 0 / 3 | Нет | Новых в выбранном контуре нет |

## Status-line Round Feed

Mirror round scores into a small state file so a live status line can show them. Refresh it whenever the table is printed and clear it when the loop finishes. Failure here must never block or alter review.

- Key the file by repository root, falling back to current working directory outside Git. Directory path: `$HOME/.config/statusline-state/loop-app-code-review/`.
- Format: one `<sev>:<score>` segment per round, joined by `;`, latest last. Start the line with `app-code|`.
- Map severity as: `c` = blocker/critical/serious, `m` = medium, `s` = minor/preference, `n` = no new in-scope findings. Example: `app-code|c:8,0;n:9,5`.

**Bash / Unix:**
```bash
RF="$HOME/.config/statusline-state/loop-app-code-review/$(printf '%s' "$(git rev-parse --show-toplevel 2>/dev/null || printf '%s' "$PWD")" | sed 's#[^A-Za-z0-9]#_#g')"
mkdir -p "$(dirname "$RF")" && printf 'app-code|%s' "c:8,0;n:9,5" > "$RF"
rm -f "$RF"
```

**Windows PowerShell:**
```powershell
$r = (git rev-parse --show-toplevel 2>$null); if (-not $r) { $r = $PWD.Path }; $RF = "$HOME/.config/statusline-state/loop-app-code-review/$($r -replace '[^A-Za-z0-9]', '_')"
New-Item -ItemType Directory -Force -Path (Split-Path $RF) | Out-Null; Set-Content -Path $RF -Value "app-code|c:8,0;n:9,5" -NoNewline
Remove-Item -Path $RF -Force -ErrorAction Ignore
```

## Reviewer Prompt Template

```text
Review this existing application independently. You have no parent conversation history. Derive findings only from repository state and tool output you inspect yourself. Stay read-only: do not edit, stage, commit, reset, stash, or push files. Do not apply fixes.

Repository: <path>

Selected severity scope (hard acceptance boundary):
- <selected levels and their exact definitions>

Selected review surfaces (hard search boundary):
- <selected surfaces and their exact definitions>

Coverage Breadth and Prior Pass Context:
- Subsystems / paths audited in prior rounds: <list of audited components or "none (first round)">
- Instructions: You are free and encouraged to re-examine previously audited files if you suspect deeper architectural, concurrency, or subtle logic defects. However, you MUST also inspect unexamined components belonging to the selected surfaces so that no blind spots remain. A score of 9.5 is prohibited if major components of the selected surface remain unvisited.

Search only selected surfaces for selected severity. Do not report unselected lower-severity findings or out-of-surface findings, reduce the score for them, or request fixes. Do not reclassify findings to fit the scope. An unmistakable incidentally discovered blocker, critical security/privacy exposure, or irreversible data-loss risk may be surfaced once as a safety override, without broadening the search.

Known findings ledger (do not rediscover these). Search for OTHER defects in the same selected scope. If you hit the same issue, emit duplicate-of: ID and do not assign a new id:
<full ledger or "empty">

Treat review as an audit handoff. Reconstruct selected journeys/surfaces, important flow, invariants, and failure behavior. Report a finding only when its concrete impact belongs to a selected level on a selected surface.

Return new in-scope findings first in severity order with file/line references and plain user impact. Clearly state when none exist. List any duplicates separately. Then explain the reconstructed flow and which components/files you inspected this round. End with a scoped score from 1 to 10: 10 when no selected-level defect remains undiscovered on selected surfaces in this pass, validation evidence was inspected, and surface coverage is complete; 9.5 when no new selected-level actionable finding remains, surface coverage is complete, and only out-of-scope, already-recorded, or subjective items may exist; below 9.5 when a new selected-level finding remains, coverage is incomplete, or required validation is missing. State the concrete new in-scope issue preventing 9.5.
```

## Final Response

- Print the final trajectory table with selected severity, selected surfaces, and exact reviewer model per round.
- State which severity levels and surfaces were reviewed and explicitly state that unselected ones were not assessed.
- Provide a clickable link to `audit-ledger.md` in the project directory where the full finding ledger is preserved. In the chat response, provide only a concise executive summary of top findings and total counts. Do not apply fixes.
- Report the scoped acceptance signal, pass count, and whether the loop passed, stopped incomplete, or was interrupted.
- Confirm model and reasoning-effort parity for every counted round.
- Report validation commands and results, safety overrides, duplicates dropped, and remaining risks.
- Remove the status-line state file after printing the final table.
