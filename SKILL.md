---
name: lite-iterative-coding
description: Use when implementing features, bug fixes, refactors, or behavior changes in an existing codebase where project rules, contracts, tests, documentation, or user intent must constrain the work
---

# Lite Iterative Coding

## Core Idea

Use the lightest spec discipline that prevents agent drift. The goal is not to produce more documents; the goal is to make intent, project facts, contracts, and verification hard to bypass.

This skill is for real development in existing codebases. It should keep small changes fast and make complex changes explicit enough to maintain.

## When to Use

Use this for:

- Feature implementation, bug fixes, behavior changes, and refactors.
- Brownfield work where existing architecture, tests, contracts, generated artifacts, storage paths, workflows, or UI states matter.
- Cross-module or cross-service changes.
- Work where the user asks to discuss design, write a plan, inspect an existing flow, or avoid free-form implementation.

Do not turn this into a heavy SDD ceremony for tiny edits. If the task is a small local change, keep the plan and documentation decisions brief.

## Required Flow

### 1. Find the Fact Sources

Before proposing or editing, locate the current project's rules and durable facts. Read only what is relevant.

Common fact sources:

- Agent instructions: `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, Copilot instructions.
- Architecture and feature docs under `docs/`, `specs/`, `architecture/`, or module-specific docs.
- API contracts, schemas, migrations, workflow configs, generated artifact conventions, storage path conventions.
- Tests that encode current behavior.
- Recent commits only when they clarify current intent or pending work.

If the project provides a structural code tool such as CodeGraph, use it for symbol relationships and impact questions. Use text search for literal strings, logs, file names, and comments.

### 2. State the Impact Chain

Before implementation, summarize the change in a short delta plan. Keep it concise and in the user's language when possible.

Use this shape:

```text
Goal:
Fact sources:
Impact:
Plan:
Verification:
Documentation:
```

For small tasks, one or two sentences are enough. For complex tasks, split the plan into 2-3 independent work units.

The impact chain should name what can break: modules, APIs, data models, state transitions, generated files, storage keys, UI rendering, background jobs, tests, or external services.

### 3. Confirm Before Editing

After stating the impact chain, stop and ask for user confirmation before making any code, test, documentation, configuration, data, or git changes.

The confirmation must be explicit execution authorization in the user's latest message. It must say to perform the change now, not only describe the desired system behavior.

Treat these as explicit authorization:

- "继续实现"
- "直接改"
- "现在实现"
- "按这个方案执行"
- "修复并提交"
- "不需要确认，直接做"

Do not treat these as authorization by themselves:

- Requirement statements: "需要...", "应该...", "我希望...", "这里要..."
- Design discussion: "这样是否可行", "你觉得...", "方案是什么"
- Scope adjustments without an execution verb: "调整一下...", "改成...", "增加一个判断..."
- Approval of an idea without execution wording: "可以", "好的", "这个方向对"

If the user asks a question, asks to discuss design, asks for analysis, or the intent is ambiguous, do not edit files or run mutating commands. Answer the question or present the proposed approach, then ask a direct confirmation question such as: "我会按这个方案修改这些文件，确认现在执行吗？"

Before every mutating operation, run this self-check:

```text
Has the latest user message explicitly authorized execution now?
Is the operation within the confirmed impact chain?
If either answer is no, stop and ask for confirmation.
```

Safe read-only actions before confirmation are allowed: inspect files, search code, read docs, run non-mutating diagnostics, and summarize findings. Avoid `apply_patch`, file writes, formatters, migrations, dependency changes, database writes, commits, pushes, or other mutating operations before confirmation.

If another active instruction encourages proactive implementation, use this stricter confirmation gate while this skill is active. If the user confirms one plan and new evidence materially changes the plan or widens scope, stop, restate the revised impact chain, and ask for confirmation again.

### 4. Maintain Project or Feature Technical Docs When Needed

Technical docs here mean long-lived project-level or feature-level explanations, not per-task change notes.

Write or update technical documentation when the task creates or materially changes:

- A lasting feature, subsystem, workflow, generation chain, parser, integration, or deployment/runtime path.
- Architecture boundaries, module responsibilities, data flow, state flow, API contracts, data models, file artifacts, storage paths, or configuration conventions.
- Behavior that future developers need to understand from a concise explanation, not only by reading code.

Do not create or update technical docs for:

- Small bug fixes with no documentation impact.
- Local UI text/style/display tweaks.
- Changes that do not alter the existing project or feature explanation.
- Cases where existing docs remain accurate.

Good technical docs describe the current system, not the history of this task. Prefer concise sections like:

```text
Purpose
Overall Flow
Key Modules
Inputs and Outputs
Contracts and Artifacts
Important Conventions
Failure Modes and Troubleshooting
Verification
```

If implementation changes the intended design, update the project or feature doc to match the final state. Do not leave docs describing the original plan after the code has evolved.

### 5. Implement the Smallest Valid Change

Start this step only after the confirmation gate above is satisfied.

Edit only files covered by the delta plan and project rules.

Avoid:

- Unrelated refactors.
- Dependency changes unless required.
- Whole-repo formatting.
- Renaming or moving files outside the task.
- Reverting user or teammate changes.

If new evidence shows the original plan is wrong, stop and restate the revised impact chain before widening scope.

### 6. Verify With Evidence

A change is not done until verification has actually run or the reason it cannot run is clear.

Use project-specific commands first. Verification may include:

- Unit or integration tests.
- Type checks, lint, format checks.
- API calls or contract checks.
- Browser inspection, screenshots, or UI state checks.
- Logs or generated artifact inspection.

Report the exact commands or checks and their result. If a check fails, fix it instead of treating it as a note. If a check is blocked by pre-existing unrelated issues, identify the blocker and run the narrowest useful verification that still proves this change.

### 7. Commit Completed Work Batches

When a coherent batch of functionality is finished and verification has passed, commit the work according to the project's Git convention. Prefer the project's documented convention if it exists. If the project has no convention, use the fallback below.

Do not commit:

- Unverified work.
- Unrelated dirty files.
- User or teammate changes that were not part of this task.
- Temporary logs, generated scratch files, or abandoned experiments.

Default branch naming:

```text
<type>/<short-description>
```

Use lowercase English words in `short-description`, joined with `-`. Common types:

| type | Use for |
|---|---|
| `feat` | New features |
| `fix` | Bug fixes |
| `refactor` | Behavior-preserving restructuring |
| `style` | UI/style-only changes |
| `docs` | Documentation updates |
| `chore` | Tooling, build, cleanup, or dependency work |

Default commit message:

```text
<type>: <subject>
<type>(<scope>): <subject>
```

Use a short subject in the user's/project's language. If a scope is used, keep it short and technical, such as `frontend`, `backend`, `api`, `parser`, or `orchestrator`.

Examples:

```text
feat: 新增文件上传批次解析类型
fix(api): 修复分页参数校验
refactor(parser): 精简规则解析入口
docs: 更新基础工程生成说明
```

Commit discipline:

- One independent feature, bug fix, or refactor per commit.
- Include documentation updates in the same commit as the code change when they describe that same feature.
- Do not push unless the user explicitly asks.
- Do not use destructive Git commands such as `git reset --hard`, `git checkout .`, or broad restore commands to clean the tree.

## Using Subagents

Use subagents when they reduce risk:

- Read-only chain analysis for unfamiliar or cross-module flows.
- Independent review of an implementation against the plan and technical docs.
- Parallel investigation of separate subsystems.

Give subagents a tight scope, allowed files, forbidden files, and expected output. Do not let subagents make broad edits unless the task has already been decomposed and boundaries are explicit.

## Common Mistakes

- Writing a long plan instead of reading the project's real facts.
- Treating markdown as success while no executable check can fail.
- Updating code but leaving feature documentation stale.
- Explaining a change as a task history instead of documenting the system's current behavior.
- Fixing nearby problems the user did not ask to fix.
- Running broad searches or reading whole modules when a structural lookup or targeted read would answer the question.
- Treating a requirement sentence such as "需要增加一个判断" as permission to edit files.
