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
