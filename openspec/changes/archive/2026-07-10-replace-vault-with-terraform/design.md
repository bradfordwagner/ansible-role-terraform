## Context

The role today (`tasks/main.yml`, `vars/main.yml`, `defaults/main.yml`) implements a generic "download HashiCorp zip release, verify checksum, unarchive, symlink" flow parameterized by `vault_version`, `vault_type` (community/enterprise), `vault_scope` (system/local), and `vault_mirror`. That flow — download → checksum-verify → unarchive → symlink — is product-agnostic and reusable as-is for Terraform; only the naming and the type dimension need to change.

Terraform's release layout at `releases.hashicorp.com/terraform` was confirmed (via `index.json` and a downloaded `1.15.8` archive) to match Vault's: `terraform_<version>_<os>_<arch>.zip`, a sibling `terraform_<version>_SHA256SUMS` file, and a single `LICENSE.txt` inside the archive. The one structural difference is that Terraform has no Enterprise CLI binary — Terraform Enterprise is a separately-distributed self-hosted app, not something downloaded from this mirror — so there is no `+ent` suffix, no second archive variant, and no per-type license files (`LA_*`, `notices`) to check for.

## Goals / Non-Goals

**Goals:**
- Re-skin the existing download/verify/unarchive/symlink flow to install Terraform instead of Vault, keeping the same `system`/`local` scope behavior (install dir, link dir, `become`) unchanged in shape.
- Remove the `vault_type` (community/enterprise) concept entirely rather than leaving unused enterprise plumbing behind.
- Keep the role dependency on `andrewrothstein.unarchivedeps` (already handles unzip deps) and the checksum-verification approach unchanged.

**Non-Goals:**
- Do not model Terraform Enterprise, Terraform Cloud, or `tfenv`-style multi-version management — this role installs exactly one pinned binary version, same as today.
- Do not change the `system`/`local` scope mechanism, the checksum algorithm (sha256), or the CI base image (`config.yaml`), all of which are already current/correct and orthogonal to the product swap.
- Do not add config file management, workspace/state handling, or any Terraform-runtime concerns — this stays a binary installer only, matching the current README's stated scope for Vault.

## Decisions

- **Drop `vault_type`/`+ent` outright instead of keeping a stub `terraform_type` var.** Terraform has nothing analogous to install, so carrying the variable forward would be dead code advertising a capability that doesn't exist. Alternative considered: keep a `terraform_type` var defaulting to `community` for symmetry with the template pattern — rejected because it invites confusion (there's no valid non-community value) and violates "don't design for hypothetical requirements."
- **Rename all `vault_*` variables/paths to `terraform_*` 1:1** (`vault_version`→`terraform_version`, `vault_scope`→`terraform_scope`, `vault_mirror`→`terraform_mirror`, `vault_install_dir`→`terraform_install_dir`, etc.), preserving the existing internal variable structure in `vars/main.yml` (minus the `_full_version`/type computation, which collapses to just using `terraform_version` directly). Alternative considered: a generic `hashicorp_product` variable to make the role multi-product — rejected as over-engineering for a single-purpose role; three near-identical lines (this role vs. a future one) beat a premature abstraction.
- **Default `terraform_version` to `1.15.8`**, the latest final (non-alpha/beta/rc) release per `releases.hashicorp.com/terraform/index.json` as of this change.
- **Collapse `test.yml` from 4 scenarios to 2`** (system scope, local scope) since the type axis is gone, and correspondingly simplify `tests/verify.yml` to drop the enterprise-only assertions (`LA_*` files, `notices` file) while keeping the `LICENSE.txt` assertion, since Terraform's archive always includes one.
- **Leave `meta/requirements.yml` and `config.yaml` untouched.** Confirmed during research that `andrewrothstein.unarchivedeps` is already pinned at its current latest tag (`3.2.0`) and the CI base image tag (`6.4.0`) is already the latest published tag with a matching `builds:` OS list — both are up to date per `CLAUDE.md`'s upgrade checklist, so touching them here would be an unrelated, unnecessary change.

## Risks / Trade-offs

- [Renaming `vault_*` vars is a breaking change for anyone already consuming this role under the old variable names] → Mitigation: no known external consumers yet (fresh template-derived repo); the role's own galaxy name is changing too, so there's no expectation of backward compatibility here.
- [Terraform's BUSL license (post-1.5.5) is still packaged as `LICENSE.txt` the same way Vault's Apache/BSL license was, so the existing single-file assertion is reused as-is] → Mitigation: verified directly by downloading and inspecting the `1.15.8` archive contents before writing this design.

## Migration Plan

Not applicable — this is a same-repo, pre-release content swap with no deployed consumers to migrate. Implementation proceeds directly via `tasks.md`; no rollback beyond reverting the commit is needed.

## Open Questions

None outstanding — Terraform's release layout, checksum format, and license-file packaging were all verified directly against `releases.hashicorp.com` before writing this design.
