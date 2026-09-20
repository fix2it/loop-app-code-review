# Loop App Code Review

An [Agent Skill](https://agentskills.io) for iterative, **read-only** review of an existing application. Findings go into a ledger; the skill does not apply patches.

Invoke as `loop-app-code-review` or `lacr`.

Variant of [`loop-code-review-3`](https://github.com/tablesguru/looop-review): same severity scale, same reviewer-parity rules, same stop criteria. The target is the existing app, not only the current task's git diff.

Before any inspection it asks (1) which severity levels to search and (2) which surfaces to search. `7` is the practical audit (surfaces 1–4). A score of 9.5 means no *new* findings in that scope — not that the app is clean.

## Install

```sh
git clone https://github.com/fix2it/loop-app-code-review.git
```

Copy the skill folder into the skills directory your agent reads, for example:

```sh
cp -R loop-app-code-review "$HOME/.claude/skills/"
# Cursor: "$HOME/.cursor/skills/"
```

If you cloned into that directory already, you are done.

## Use

```text
loop-app-code-review
lacr
```

It first asks how serious a bug must be to count, then which parts of the app to search. After that the review starts. It never changes your project files.

## Attribution and license

The review-loop comes from Dima Sukharev's MIT [`loop-code-review-skill`](https://github.com/di-sukharev/loop-code-review-skill) and tablesguru's [`loop-code-review-3`](https://github.com/tablesguru/looop-review).

This variant was written by Aleks ([@fix2it](https://github.com/fix2it)). See [NOTICE.md](NOTICE.md).

Licensed under the [MIT License](LICENSE).
