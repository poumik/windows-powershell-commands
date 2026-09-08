# AGENTS.md — Agent Instructions for powershell-commands

Guidance for AI coding agents (Copilot, Codex, Claude, OpenCode, etc.) maintaining this repository. Read this before making any change.

## Project Overview

- **What it is:** A documentation-only repository — a curated quick-reference guide of Windows 11 PowerShell commands for system administration and management.
- **Primary artifact:** [`powershell-commands.md`](powershell-commands.md) — a single, self-contained Markdown document with 18 numbered sections of categorized command snippets.
- **Supporting files:**
  - [`README.md`](README.md) — repo landing page: getting started, linked table of contents, license, disclaimer.
  - [`LICENSE`](LICENSE) — MIT, copyright poumik.
  - `opencode.json` — config for the OpenCode agent tool (web-search MCP server). Not part of the documentation itself; don't edit unless asked.
- **There is no code to build, no tests to run, and no dependencies.** All work is Markdown editing and fact verification.

## Repository Conventions

### Document structure of `powershell-commands.md`

- Sections are numbered (`## 1. ...` through `## 18. ...`) and separated by `---` rules.
- Every section begins with a `> **Permission:**` blockquote tag: **Administrator**, **None (read-only)**, or **Mixed** (queries read-only, changes need elevation). Keep these tags accurate when adding or changing commands.
- Each section heading contains an anchor link; the README table of contents links to them. If you rename a section, update the README links too.
- The document has an internal **Contents** list (near the top) and a **Quick Reference** table — update both when adding a new section.
- A top-level note explains PowerShell 5.1 vs 7+ compatibility. Commands that only exist in PowerShell 7+ must be marked **(PS 7+)** in their comment.
- Code fences use ` ```powershell ` (or plain ` ``` ` where appropriate). Keep one comment per command, in the established style.

### Style rules

- **One command (or one tightly related group) per commented snippet.** Comments describe what the command does, not how PowerShell works.
- Prefer single-purpose, copy-pasteable snippets over multi-step scripts.
- Placeholder values in examples: `<name>`, `"Program Name"`, `"MyTask"`, `"PC01"`, `EXPECTED_HASH`.
- Use native cmdlet parameters over `Where-Object` filtering when one exists (e.g. `Get-NetFirewallRule -Enabled True`).
- Document third-party tools explicitly as such (e.g. PSWindowsUpdate is from the PowerShell Gallery, not built in).

### Editing rules

- **Do not restructure or renumber sections** unless the user explicitly asks. Section anchors are linked from README and the internal Contents.
- When adding a section, continue the numbering and update: the internal Contents list, the README table, and the Quick Reference table (if the new section has an obvious headline command).
- Keep the intro compatibility note and permission-tag legend in sync with reality.

## Fact-Verification Requirements (critical)

This repo's value is **correctness**. Before adding or changing a command, verify it against authoritative sources:

1. **Prefer official Microsoft Learn documentation** (`learn.microsoft.com`) for cmdlet syntax, parameters, and elevation requirements. Cmdlet reference pages: `https://learn.microsoft.com/en-us/powershell/module/<module>/<cmdlet>`.
2. **Verify claims about Windows versions.** Deprecations and removals (e.g. "X removed in Windows 11 24H2") change over time — check the current [Deprecated features](https://learn.microsoft.com/en-us/windows/whats-new/deprecated-features) and [Removed features](https://learn.microsoft.com/en-us/windows/whats-new/removed-features) pages before writing such statements. Do not state that something was removed from a Windows version unless Microsoft documents it.
3. **Verify cmdlet names and parameters exactly.** They are easy to get wrong (e.g. `BackupToAAD-BitLockerKeyProtector`, not `BackupToAAD-BitLocker`; it also requires `-MountPoint`). Fetch the cmdlet's reference page when unsure.
4. **Check elevation requirements per command, not per section.** E.g. `Clear-DnsClientCache` (≈ `ipconfig /flushdns`) and `powercfg /batteryreport`, `/energy`, `/list` run unelevated, while `powercfg /systemsleepdiagnostics` requires elevation. When docs don't state it clearly, mark it **Mixed** with a short explanation rather than guessing.
5. **Cross-check winget flags** (`--silent`, `--accept-source-agreements`, `--accept-package-agreements`, `--all`) against `winget --help` output or Microsoft's winget docs if there's any doubt.
6. If web access is unavailable and a fact can't be verified, **flag the uncertainty to the user** instead of asserting it.

### Known corrections already applied (don't regress)

- `BackupToAAD-BitLockerKeyProtector` with `-MountPoint` (the old text had a nonexistent cmdlet name).
- `Microsoft.PowerShell.LocalAccounts` is **not** removed in Windows 11 24H2 — the `net user` commands are just classic equivalents.
- `Clear-DnsClientCache` runs unelevated.
- `powercfg` reports: `/batteryreport`, `/energy`, `/list` run unelevated.

## Workflow

1. Read [`powershell-commands.md`](powershell-commands.md) and the relevant section fully before editing.
2. Verify facts (see above) using available tools — fetching official docs pages is preferred over memory.
3. Make the smallest change that is correct; keep formatting consistent with surrounding content.
4. After editing, re-check: internal Contents list, README table links, Quick Reference table, permission tags, and **(PS 7+)** markers.
5. Report to the user what was verified and what was changed, citing sources for factual claims.

## Out of Scope

- Do not turn the guide into a tutorial/course (it's a quick reference).
- Do not add build tooling, linters, scripts, or CI — the repo is intentionally plain Markdown.
- Do not change the license, copyright, or `opencode.json` unless explicitly asked.
- Do not add new sections/categories without user request.