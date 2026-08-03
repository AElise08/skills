---
name: sol-luna-workflow
description: Use when a software task should be architected and coordinated by Sol, with implementation delegated to Luna Max. Sol owns requirements, decomposition, interfaces, verification, and acceptance; Luna Max owns bounded code changes. Trigger for feature work, bug fixes, refactors, migrations, and other repository changes where a separate implementation lane and fresh review improve reliability.
---

# Sol → Luna Max Workflow

## Overview

Use Sol as the primary architect/orchestrator and route implementation to Luna Max through a native custom-agent thread. Sol defines the contract and verifies the result; Luna Max writes the bounded implementation. Do not claim completion until repository checks pass and Sol has reviewed the actual diff.

## Workflow

### 1. Establish the contract

Before delegating, inspect the repository and write a compact implementation spec containing:

- objective and acceptance criteria;
- files/directories Luna Max may own;
- interfaces, data contracts, and constraints;
- verification commands and expected evidence;
- risks, assumptions, and explicit non-goals.

Do not delegate an underspecified request. Resolve ambiguity from the codebase when safe; otherwise state the decision in the spec.

### 2. Verify routing before implementation

Use native subagents/custom agents only. Confirm that the selected implementation thread is Luna Max (exact configured agent type and, when exposed, model/reasoning profile). If Luna Max is unavailable or metadata is ambiguous, stop the implementation lane and report the actionable setup gap; never silently substitute another model.

Preferred roles:

- Orchestrator: Sol, high reasoning;
- Implementation: Luna Max, using the repository's configured Luna Max custom-agent profile;
- Review: a fresh Sol read-only reviewer, when available.

The primary Sol session remains responsible for architecture, routing, parent verification, and acceptance. Luna Max receives only the task-local context and the written spec needed to implement it.

### 3. Delegate bounded implementation

Ask Luna Max to:

- implement only the owned files and requested behavior;
- preserve unrelated user changes;
- run the specified tests, lint, build, or focused checks;
- report changed files, commands run, failures, and remaining uncertainty.

Do not let the implementation agent redefine the architecture or broaden scope without returning the decision to Sol.

### 4. Verify and review

After Luna Max returns, Sol independently inspects the actual diff and reruns relevant verification. Check behavior, scope, regressions, security-sensitive changes, and acceptance criteria—not merely the implementer's summary.

If a fresh Sol reviewer is available, request a read-only review of the final diff and evidence. The reviewer must return one of:

- `ship`: acceptance criteria met and evidence sufficient;
- `fix-first`: concrete defects or missing verification;
- `rethink`: architecture or scope is wrong.

Only report completion after `ship`. On `fix-first`, return a focused correction to Luna Max, then repeat verification and review. On `rethink`, stop and revise the spec with the user or from newly discovered repository evidence.

## Safety and scope

- Never overwrite unrelated work or secrets.
- Do not expose environment variables, tokens, private prompts, or rollout contents in reports.
- Treat a reviewer as read-only; if the reviewer mutates files or read-only isolation cannot be established when required, stop and report the residual risk.
- Preserve repository conventions and use the project's own verification commands.
- Prefer small, reviewable delegation units over one broad implementation request.

## Minimal delegation prompt

Use this shape when spawning Luna Max:

> Implement the bounded task below. Do not change files outside the ownership list or redesign the architecture. Preserve unrelated changes. Run the listed checks and report exact results. If the spec is insufficient or a broader change is required, stop and return the question to Sol.
>
> Objective: …
> Acceptance criteria: …
> Owned files: …
> Interfaces/constraints: …
> Verification: …
