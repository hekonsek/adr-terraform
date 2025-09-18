# Apply `terraform fmt` after changing project

## Context

Terraform source files drift quickly when different contributors use different editors or whitespace settings. Formatting drift makes diffs noisy, triggers lint failures in CI, and forces reviewers to parse unrelated formatting changes. We want the repo to stay consistently formatted without relying on maintainers to fix whitespace after the fact.

## Decision

Whenever a change touches any `*.tf` or `*.tfvars` file, run `terraform fmt -recursive` from the repository root before opening a pull request. This applies to both manual edits and generated code. You can wire the command into pre-commit hooks or task runners, but the contributor is responsible for ensuring formatting is clean in the final commit.

## Consequences

**Positive**

- Consistent Terraform formatting keeps diffs readable and focused on behavioral changes.
- CI pipelines avoid avoidable failures caused by whitespace or ordering drift.
- Reviewers do not need to chase contributors for formatting follow-ups.

**Negative**

- Adds a local step (or tooling setup) to the developer workflow.
- Requires contributors to have Terraform CLI available even for small formatting-only fixes.

## Alternatives considered

- **Rely on CI to fail and maintainers to reformat:** rejected because it slows down merges and shifts work to the maintainer.
- **Auto-format via server-side hooks only:** reduces local burden but hides feedback until after push; we prefer immediate local feedback.
