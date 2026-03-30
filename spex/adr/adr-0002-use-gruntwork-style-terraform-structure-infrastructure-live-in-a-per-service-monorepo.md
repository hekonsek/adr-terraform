# Use Gruntwork-style Terraform structure (Infrastructure Live) in a per-service monorepo

## Context

We want all Terraform for a service in one place for:
- Simpler discovery, onboarding, and atomic changes across environments
- Clear separation between reusable modules and environment compositions
- Safe promotion (dev→staging→prod)

## Decision

Adopt a [Gruntwork-style "Infrastructure Live" layout](https://docs.gruntwork.io/2.0/docs/overview/concepts/infrastructure-live) inside a single repository:

```
environments/ # environment compositions (what we run)
  dev/ 
    gcp/
      vpc/
      iam/                 
  staging/
  prod/
modules/ # reusable versioned modules (how we build)
  gcp/
    iam/
    vpc/
    ecs-service/
    rds/
```

If you create new Terraform structure in existing project non-Terraform project (for example web application), don't create `environments` and `modules` directories in root folder. Instead create `iac` directory in root which in turn contains `environments` and `modules` directories.

You can include region subdirectories for non-global cloud providers like AWS. For example: /aws/eu-central-1/

Modules are semver-tagged via git tags (e.g., `modules/networking/vpc@v1.4.2`) and pinned from `live/` using `?ref=<tag>`.

Minimal example usage from `environments`:

```hcl
module "vpc" {
  source = "git::ssh://git.example.com/infra.git//iac/modules/gcp/vpc?ref=v1.4.2"
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