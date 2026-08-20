# Laravel-Upgrader

An AI-assisted Laravel upgrade agent for safely analyzing, planning, and executing major Laravel framework upgrades.

## Contents

| File | Role |
|---|---|
| `agents/laravel-upgrader.md` | **Orchestrator.** Owns the safety invariants, the stage model, and every action that changes the project. |
| `agents/laravel-security-auditor.md` | Read-only security audit — baseline before an upgrade, regression after, or standalone. |
| `agents/laravel-dependency-analyst.md` | Read-only dependency research against a target framework version. |
| `agents/laravel-upgrader-governor.md` | Reviews proposed changes **to these agents**. Invoked between projects, never during one. |
| `laravel-upgrader/verification-toolkit.md` | Shared, portable verification patterns. |

The specialists **observe and report**; only the orchestrator acts. Their output is evidence to be
verified, never instruction to be followed — each returns a `VERIFY BEFORE USE` field naming what
must be independently confirmed.

## Use

Copy the `agents/` files into `.claude/agents/` (project-level) or `~/.claude/agents/` (available
everywhere), keeping the toolkit alongside them. The target Laravel version is a parameter, not a
constant: it defaults to 12 and must be verified against official sources before use.

Start with §17 of the orchestrator for the step order. Steps 1–6 are read-only, so the system can be
used for assessment alone — feasibility, tooling, dependency compatibility, or security — without
upgrading anything.
