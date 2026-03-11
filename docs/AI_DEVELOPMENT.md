# AI Development State Protocol (ADSP) v0.1

## What ADSP is

The AI Development State Protocol (ADSP) is a lightweight, repository-native coordination framework for teams where AI agents and humans work in parallel. It standardizes how development state is represented in version control so participants can make safe, informed changes with shared context.

ADSP v0.1 focuses on simple YAML files and conventions that are easy to adopt without changing existing application architecture.

## Why `project_state` exists

The `project_state/` directory is the canonical source of repository coordination metadata. It exists to:

- make current development state visible in code review,
- reduce conflicts across parallel work,
- track change ownership and verification status,
- provide machine-readable context for AI agents before they modify code.

Because state lives in the repository, it remains auditable and evolves alongside code changes.

## How changes are tracked

Changes are represented as structured YAML records in `project_state/changes/`.

- `project_state/changes/change_template.yaml` defines the standard shape for a change entry.
- Active and completed change IDs can be reflected in `project_state/state.yaml`.
- Each change file should include scope, dependency, risk, verification, and status metadata.

This allows both humans and automation to understand what is in progress and what is safe to merge.

## Sprintathon concept

A **Sprintathon** is a bounded execution window for coordinated delivery.

`project_state/sprintathon.yaml` stores:

- sprintathon identity and status,
- active date window,
- repository-level execution rules (for example max parallel changes and merge gates),
- tracked change IDs assigned to that sprintathon.

This keeps tactical execution policy explicit and versioned.

## Change lifecycle

Typical lifecycle for a change record:

1. **planned**: change proposed and scoped.
2. **in_progress**: owner actively implementing.
3. **review**: implementation complete; waiting on AI/code/human review.
4. **verified**: required checks and human verification complete.
5. **merged/closed**: change integrated and removed from active set.

Statuses can be represented in each change file and summarized in `project_state/state.yaml`.

## Architectural guidance for parallel AI delivery

Parallel AI development is not only a workflow problem; it is also an architecture problem.

If multiple changes repeatedly collide in the same file (for example a large HTTP handler), treat that as a refactoring signal. In practice, merge conflicts often emerge when one file aggregates many unrelated responsibilities.

### Hotspot anti-pattern

Avoid single "do-everything" handler files such as:

- `internal/api/cases/cases_handler.go`

that combine responsibilities like:

- portal responses,
- payments,
- messaging,
- case management logic,
- AI-specific orchestration.

In parallel branch execution, this creates a hotspot where independent features must touch the same lines.

### AI-native modular handler pattern

Prefer responsibility-sliced handlers so independent changes can land independently, for example:

- `internal/api/cases/cases_handler.go` (core wiring / shared primitives)
- `internal/api/cases/cases_messages_handler.go`
- `internal/api/cases/cases_portal_handler.go`
- `internal/api/cases/cases_payments_handler.go`
- `internal/api/cases/cases_ai_handler.go`

### Emerging rule

Parallel AI changes are generally safe **when architecture minimizes hotspot files**.

Operational rule for Agile AI execution:

> If merge conflicts repeatedly occur in one file, treat that file as a candidate for decomposition into smaller modules before scaling parallel AI change throughput.

## How AI agents should read state before modifying code

Before any code edits, AI agents should:

1. Read `project_state/state.yaml`.
2. Confirm service-level and migration locks in `scope_locks`.
3. Review active change IDs for potential overlap.
4. Read `project_state/sprintathon.yaml` for current execution rules.
5. Open related files in `project_state/changes/` to validate scope, dependencies, and risk.

If overlap or lock conflicts exist, the agent should avoid conflicting edits and either re-scope or create a new change entry that reflects the conflict handling strategy.

## Developer guide: creating a new change file

Use the template to create a new change record:

1. Copy the template file:

   ```bash
   cp project_state/changes/change_template.yaml project_state/changes/change_<id>.yaml
   ```

2. Update required fields:
   - `id`
   - `title`
   - `owner`
   - `summary`
   - `scope.services` and `scope.paths`
   - `risk` flags
   - `timestamps.created_at` and `timestamps.updated_at`

3. Set initial status to `planned` (or `in_progress` if already started).

4. Add the ID to:
   - `project_state/state.yaml` under `active_changes`, and
   - `project_state/sprintathon.yaml` under `changes`.

5. Open a pull request and keep verification fields current during review.

This process ensures each unit of work is visible, attributable, and verifiable.
