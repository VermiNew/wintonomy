# Agent Instructions

## Project Overview

**Wintonomy** (Windows + Autonomy) — interactive Windows setup automation that gives users control over fresh Windows installations.

Legacy reference: `.agents\_legacy-windows11-setup\` — read-only reference for understanding behavior, architecture, naming, edge cases. Do not copy whole modules or large sections.

## Workflow: Iterative Prototyping

Never do a "waterfall" — never write large amounts of code in a single step.
Follow this cycle for every piece of work:

### Cycle

1. **Plan** — Analyze the task. Define a small, concrete mini-step.
2. **Pick the simplest part** — Start with the easiest piece. Build on solid ground.
3. **Write code** — Implement only that small piece.
4. **Read your own code** — Review what you wrote before running anything.
5. **Verify** — Run the verification pipeline appropriate for PowerShell:
   - **Syntax** — `pwsh -NoProfile -Command "[void][System.Management.Automation.Language.Parser]::ParseFile('<path>', [ref]`$null, [ref]`$null)"` — parses the file without executing it.
   - **Module import** — `pwsh -NoProfile -Command "Import-Module .\modules\<name>.psm1 -Force; Get-Command -Module <name>"` — verifies the module loads and lists its exports.
   - **PSScriptAnalyzer** (if available) — `Invoke-ScriptAnalyzer -Path .\<file>.ps1 -Recurse` (only run if the module is already installed; do not install it without permission).
   - **BOM check** — every `.ps1`/`.psm1` must start with a UTF-8 BOM (`EF BB BF`). Check the first three bytes.
   - **Manual testing** — **DO NOT run** system-configuring scripts automatically. They modify the registry, install packages, create backups, create restore points, etc. The **user** tests changes manually.
   - Exception: pure functions (e.g. text-formatting helpers) may be invoked in isolation in `pwsh` if they have no side effects.
6. **Fix or commit:**
   - If something is broken → fix it and go back to step 5.
   - If everything works → commit with a conventional commit message.
7. **Repeat** — Pick the next small piece. Improve the prototype toward the final version.

### Key Principles

- **A commit is a reward for working code**, not for written code.
- **Every commit must be a working state** of the application.
- **Verification is the heart of the process** — code that compiles but looks wrong is still broken.
- **Small steps prevent compounding errors** — never build on an unverified foundation.

## PowerShell-specific rules

- **UTF-8 with BOM** — save every `.ps1`/`.psm1` as UTF-8 with BOM (`EF BB BF`). Without a BOM, PowerShell 5.1 reads them as Windows-1250 and Polish characters / Unicode box-drawing glyphs get mangled.
- **Idempotency** — every step must first check whether the work is already done before modifying anything.
- **Y/n confirmation** — every sub-step asks the user via `Confirm-*` before performing a destructive action.
- **Logging** — use `Write-Log`, `Write-Step`, `Write-OK`, `Write-Skip`, `Write-Warn`, `Write-Fail` from `helpers.psm1`. Do not use raw `Write-Host` outside helpers if a helper produces the same effect.
- **`#Requires`** — scripts and modules require `#Requires -Version 7`.
- **Function naming** — use approved verbs (`Get-Verb`).
- **No reboots** — never insert `Restart-Computer`. The script informs the user that a reboot is needed and exits.
- **Don't `winget install` packages that are not in the repo** — run `winget search` before adding a new ID. If the package is not in winget, download the `.exe`/`.zip` directly.

## Commit Convention

Use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

### Format

```plaintext
<type>(scope): <description>
```

### Types

- `feat` — new feature
- `fix` — bug fix
- `style` — visual/CSS changes (not code style)
- `refactor` — code restructuring without behavior change
- `docs` — documentation
- `chore` — tooling, dependencies, config
- `test` — adding or updating tests

## Safety & Communication

- **Never `git add .` or `git add -A`** — always review changed files first (`git status`, `git diff`) and stage only the files relevant to the current task.
- **Be careful and deliberate** — double-check actions before executing them, especially destructive ones.
- **Ask before deleting** — never remove files, components, or significant code without asking the user first.
- **Discuss before acting** — explain what you plan to do and wait for confirmation. Exception: trivial, low-risk changes (e.g., fixing a typo) can proceed without asking.

## Code Hygiene

- **No new dependencies without asking** — never add new `winget` packages, PowerShell modules (`Install-Module`), or scripts from the internet without the user's approval.
- **Comments are welcome** — add meaningful comments where they help understand the code; avoid obvious or redundant ones.
- **Don't refactor unrelated code** — only touch code that is part of the current task. Don't "improve" or reorganize things that already work.
- **Don't write tests unless asked** — you may suggest writing tests, but don't create them on your own.
- **Read before writing** — always read existing code before modifying it. Understand conventions, patterns, and context first.
- **Don't over-engineer** — do exactly what was asked. Don't add extra features, abstractions, or "improvements" that weren't requested.

## Project Language

- UI language: Polish
- Code (variables, components, comments): English
