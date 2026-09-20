# Loop App Code Review

An [Agent Skill](https://agentskills.io) for iterative, **read-only** code audit and review of an existing application.

Unlike git-diff review tools that patch code on the fly, `loop-app-code-review` performs a systematic audit across user-selected surfaces, maintains an ongoing finding ledger in `audit-ledger.md`, and **never touches your source code**.

Invoke as `loop-app-code-review`, `/lacr`, or `lacr`.

---

## Key Features

- **Read-Only Audit:** Collects defects into `audit-ledger.md` without editing, committing, or breaking your project files.
- **Two-Axis Scoping:** Orthogonal boundaries for **Severity** (1–7) and **Review Surfaces** (1–7) to eliminate noise and keep focus on real bugs.
- **Coverage Breadth Contract:** Reviewers are encouraged to re-examine critical files for subtle bugs, but a passing score of 9.5/10 is strictly prohibited until all major paths of the selected surface have been visited at least once.
- **Fast-Path Support:** Run instantly in a single command (`/lacr 1-3 7`) or use the interactive step-by-step questionnaire.
- **Aggregated Tool Findings:** Homogeneous static analysis and linter errors are grouped cleanly rather than spamming the ledger.
- **Cross-Platform:** Tested for **macOS / Linux** (Bash) and **Windows** (PowerShell 5.1 & 7+).

---

## Supported AI Environments

| AI Assistant / Environment | Installation Directory |
| :--- | :--- |
| **Claude Code** | `~/.claude/skills/loop-app-code-review/` |
| **Google Antigravity / Gemini** | `~/.gemini/config/skills/loop-app-code-review/` |
| **Cursor** | `~/.cursor/skills/loop-app-code-review/` |
| **OpenAI Codex** | `~/.codex/skills/loop-app-code-review/` |
| **Universal Agent Skills** | Any environment compliant with [agentskills.io](https://agentskills.io) |

---

## Installation

Clone the repository:

```sh
git clone https://github.com/fix2it/loop-app-code-review.git
```

Copy into your preferred agent's skills directory:

```sh
# Claude Code:
cp -R loop-app-code-review "$HOME/.claude/skills/"

# Google Antigravity / Gemini CLI:
cp -R loop-app-code-review "$HOME/.gemini/config/skills/"

# Cursor:
cp -R loop-app-code-review "$HOME/.cursor/skills/"

# OpenAI Codex:
cp -R loop-app-code-review "$HOME/.codex/skills/"
```

---

## Usage

### 1. Fast-Path (Recommended for Everyday Use)
Specify severity and surface upfront to start immediately without follow-up questions:

```text
/lacr 1-3 7
```
*(Audit levels 1–3 [Blockers, Critical, Serious] on surface 7 [Practical Audit: primary journey, dependencies, trust boundaries, and failure paths]).*

### 2. Interactive Mode
Simply invoke the command and the agent will guide you through severity and surface selection:

```text
/lacr
# or
loop-app-code-review
```

1. **Step 1:** Select severity levels (`1-3`, `1-4`, or `7` for all).
2. **Step 2:** Select surfaces to audit (`7` is practical audit ~80% of real bugs, `1-2` for smoke check, `6` for full walk).
3. **Execution:** The agent runs fresh, independent reviewer passes, updates `audit-ledger.md` in your project root, and prints a compact trajectory scoreboard in chat until the scope is fully verified.

---

## Output

- **In Chat:** A clean, compact Score Trajectory table showing round-by-round progress and the link to the ledger.
- **In Project:** A persistent [`audit-ledger.md`](audit-ledger.md) file containing every verified in-scope finding with file/line references, symptoms, and impact descriptions.

---

## Attribution and License

The review-loop architecture is adapted from Dima Sukharev's MIT [`loop-code-review-skill`](https://github.com/di-sukharev/loop-code-review-skill) and tablesguru's [`loop-code-review-3`](https://github.com/tablesguru/looop-review).

This app audit variant was created by Aleks ([@fix2it](https://github.com/fix2it)). See [NOTICE.md](NOTICE.md).

Licensed under the [MIT License](LICENSE).
