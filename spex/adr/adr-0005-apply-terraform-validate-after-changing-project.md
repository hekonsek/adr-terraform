# Apply `terraform validate` after changing project

## Context

Terraform changes can introduce syntax errors, missing variables, or invalid references that only surface when validation runs. When contributors skip local validation, CI catches the errors late, causing failed pipelines and slow feedback. We want fast, local confirmation that Terraform code is valid before review.

## Decision

Whenever a change touches any `*.tf` or `*.tfvars` file in a project, run `terraform validate` from the project root (after `terraform init` if needed) before opening a pull request. This applies to both manual edits and generated code. Contributors are responsible for ensuring validation passes.

## Consequences

**Positive**

- Catches configuration errors before CI, shortening feedback loops.
- Reduces failed pipelines and review churn caused by basic validation failures.
- Encourages consistent use of the Terraform CLI in local workflows.

**Negative**

- Requires running `terraform init` at least once to fetch providers.
- Adds a local step to the workflow, which can be slow for large projects.

## Alternatives considered

- **Validate only in CI:** rejected because feedback arrives late and blocks merges.
- **Rely on `terraform plan` alone:** rejected because validation is faster and can run without backend access.
