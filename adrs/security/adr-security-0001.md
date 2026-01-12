# Do Not Use Terrascan as Security Scanner for Terraform Configurations

## Context

Terrascan has historically been used as a static analysis security scanner for Infrastructure as Code (IaC), including Terraform configurations. It provided policy-based checks for cloud misconfigurations and security risks and was commonly integrated into CI/CD pipelines and GitHub Actions.

However, multiple independent signals indicate that Terrascan is no longer actively maintained by its original maintainer (Tenable / Accurics):

* The official Terrascan GitHub repository has been **archived and marked read-only** (November 2025), preventing further upstream development or community contributions.
* There have been **no recent releases** and numerous open issues without maintainer response.
* **Official end-of-support announcements** from Tenable indicate Terrascan has been sunsetted within Tenable products and customers are advised to migrate to other solutions.
* Documentation and policy updates have stalled, increasing the risk of incompatibility with newer Terraform versions, providers, and cloud features.

As a result, continuing to rely on Terrascan introduces operational and security risks for long-lived Terraform codebases.

## Decision

Do not use Terrascan as a security scanner for Terraform configurations.

Terrascan is considered effectively discontinued and will not be adopted or extended in current or future Terraform-based projects. Existing pipelines should be migrated away from Terrascan.

The decision is based on the following factors:

* **Lack of active maintenance**: Archived repositories and lack of releases mean no fixes for bugs, false positives, or new vulnerabilities.
* **Security risk**: Unmaintained security tooling can create a false sense of safety and may miss newly emerging misconfiguration patterns.
* **Terraform ecosystem evolution**: Terraform language features, providers, and cloud services evolve rapidly and require continuous rule updates.
* **Operational risk**: CI/CD failures or incorrect results may occur as Terrascan becomes incompatible with newer Terraform versions.
* **Vendor direction**: Tenable has explicitly moved away from Terrascan in favor of other commercial offerings.

## Consequences

Positive:

* Reduced risk of relying on obsolete or inaccurate security tooling.
* Clear guidance for teams evaluating IaC security scanners.
* Opportunity to standardize on actively maintained tools with stronger community or vendor support.

Negative:

* Migration effort required for existing CI/CD pipelines using Terrascan.
* Requirement for adopting another tools providing similar functionalities.

## Alternatives Considered

- **Continue using Terrascan**. Rejected due to lack of maintenance, archived upstream repository, and end-of-support status.
- **Fork and self-maintain Terrascan**. Would require significant long-term engineering effort. No clear community momentum around a maintained fork. High maintenance burden compared to adopting modern alternatives.
- **Use actively maintained IaC security scanners**. Accepted, see further ADRs.
