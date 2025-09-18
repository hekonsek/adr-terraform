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