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
