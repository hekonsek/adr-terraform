# Follow idiomatic Terraform module structure  

## Context

## Decision

We recommend to follow idiomatic Terraform module structure:

📂 Inside a reusable Terraform module
- `main.tf` → Core resources and data sources (the logic of the module).
- `variables.tf` → Input variables with types, descriptions, and defaults.
- `outputs.tf` → Output values that the root module can consume.
- `versions.tf` → The terraform {} block with:
    - required_version (Terraform version constraint).
    - required_providers (provider version constraints).
- `providers.tf` (optional) → Sometimes used to declare provider configurations if the module needs multiple providers (but usually provider config comes from the root, not the module).
- `locals.tf` (optional) → Local variables for intermediate values or computed expressions.
- `data.tf` (optional) → Data sources, if separated for clarity.
- `README.md` → Documentation for how to use the module.
- `examples/` → Example usage of the module (best practice for reusable, published modules).
- `tests/` → Contains automated tests for the module. Typically, this includes integration tests written in Go using [Terratest](https://terratest.gruntwork.io/), which deploy the module in a real or test environment to verify its behavior. The folder may also include test fixtures, helper scripts, and configuration files needed to run the tests.

📂 In a root module (where you use the reusable modules)
- main.tf → Instantiates modules and defines resources.
- providers.tf → Provider configuration (region, credentials, etc.).
- versions.tf → Terraform and provider constraints.
- variables.tf → Input variables for the root configuration.
- outputs.tf → Outputs exposed from the root.
- terraform.tfvars / *.auto.tfvars → Values for variables (e.g., environment-specific).

## Consequences

- If this structure is not followed in the project, suggest to start doing so.


# Use Gruntwork-style Terraform structure (Infrastructure Live) in a per-service monorepo

## Context

We want all Terraform for a service in one place for:
- Simpler discovery, onboarding, and atomic changes across environments
- Clear separation between reusable modules and environment compositions
- Safe promotion (dev→staging→prod)

## Decision

Adopt a [Gruntwork-style "Infrastructure Live" layout](https://docs.gruntwork.io/2.0/docs/overview/concepts/infrastructure-live) inside a single repository:

```
live/                     # environment compositions (what we run)
  envs/
    dev/                  # You can include region subdirectories for non-global cloud providers like AWS. For example: /dev/eu-central-1/
    staging
    prod
  globals/                # Optional: environment-independent: some of IAMs, DNS, SCPs
  policies/               # Optional: OPA/Sentinel/Conftest
modules/                  # reusable versioned modules (how we build)
  networking/vpc/
  compute/ecs-service/
  data/rds/
tooling/                  # Optional: CI workflows, shared scripts, lint/policy configs
```

**Modules** are semver-tagged via git tags (e.g., `modules/networking/vpc@v1.4.2`) and **pinned** from `live/` using `?ref=<tag>`. It is fine to keep `modules/` directory without grouping for smaller deployments:

```
modules/
  vpc/
  ecs-service/
  rds/
```

Minimal example usage from `live`:

```hcl
module "vpc" {
  source = "git::ssh://git.example.com/infra.git//terraform/modules/networking/vpc?ref=v1.4.2"
  name   = "core-prod-euc1"
  cidr   = "10.20.0.0/16"
  tags   = local.standard_tags
}
```

## Consequences

**Positive**

* Single source of truth; easier cross-env changes and reviews.
* Clear separation of concerns (live vs. modules) within one repo.
* Whole IaC history is tracked within one git repository history
* Deterministic promotion via pinned module tags.
* Simplicity of discovery/onboarding.
* Repeatable promotion across environments.
* Minimize repo sprawl.

**Negative**

* Release choreography inside one repo (tag modules, then bump in live).
* Larger repo; requires discipline on boundaries and CI performance.

## Alternatives considered

* **Two-repo (live + modules):** cleaner ownership lines, but splits change history and adds coordination overhead. Overkill for Infrastructure Live per-service approach.

# Don't refactor terraform-docs generated documentation

## Context

Our modules embed generated documentation snippets between `<!-- BEGIN_TF_DOCS -->` and `<!-- END_TF_DOCS -->` comments. These sections come from [terraform-docs](https://github.com/terraform-docs/terraform-docs) during local workflows or CI pipelines. Contributors sometimes notice formatting issues or outdated content inside these blocks and attempt to "clean them up" manually.

## Decision

Treat all content between the terraform-docs markers as generated output and leave it untouched. When documentation needs to change, update the underlying Terraform configuration and let CI pipeline update the documentation. Manually refactoring autogenerated docs only generates extra review effort for person accepting your changes.

## Consequences

**Positive**

- Keeps generated documentation in sync with module inputs/outputs without drift.
- Avoids noisy diffs and review churn caused by manual tweaks that will be overwritten.
- Reinforces a consistent workflow: update code, then regenerate docs.

**Negative**

- Contributors should know how that terraform-docs is CI generated.

## Alternatives considered

- **Manual edits of generated markdown:** rejected because CI regeneration would overwrite the changes and waste reviewer effort.
- **Local generation before commit:** possible, but discouraged. Relying on contributors to run terraform-docs locally can lead to inconsistencies and missed steps. We prefer CI automation to ensure documentation is always up-to-date and standardized.

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


# Always use remote state

## Context

Terraform state is the source of truth for what is actually managed in each environment. When state is stored locally, it is easy to lose, duplicate, or diverge between contributors and CI. That leads to drift, accidental resource recreation, leaking secrets, and conflicts when multiple people run `terraform apply`. Remote backends provide shared access, locking, encryption, versioning, and auditability that are required for safe collaboration.

## Decision

All root modules in this repository must use a remote backend for state storage. Remote state must support locking and encryption (for example, a managed remote backend with built-in locking, or object storage paired with a lock table). Backend configuration is part of the root module configuration, while sensitive backend parameters are provided via environment-specific config files or CI variables.

Local state is not allowed for shared environments (dev/staging/prod) or any infrastructure that is expected to be managed by more than one person or pipeline.

## Consequences

**Positive**

- Single, authoritative state for each environment, enabling collaboration and CI/CD.
- State locking reduces concurrent apply conflicts and corruption risk.
- Versioning and backups make recovery from mistakes and outages possible.
- Access control and audit logs are centralized instead of per-developer laptops.

**Negative**

- Requires provisioning and maintaining a backend (and credentials) before first apply.
- Backend outages or access issues can block plan/apply operations.
- Changing backend configuration requires re-initialization and state migration.

## Alternatives considered

- **Local state per contributor:** rejected because it causes state divergence and unsafe concurrent changes.
- **Remote state without locking:** rejected because it still allows parallel writes and corruption.
