## Why

This repository was bootstrapped from the `bradfordwagner.ansible.role.*` GitHub template, which ships a HashiCorp Vault installer as its worked example. The repo (and its `origin` remote, `ansible-role-terraform`) is actually meant to install Terraform, not Vault, so every template-inherited file still describes the wrong product.

## What Changes

- Rename the role from a Vault binary installer to a Terraform binary installer: `tasks/main.yml`, `vars/main.yml`, `defaults/main.yml`, `meta/main.yml`, `README.md`, `test.yml`, `tests/verify.yml` all currently reference `vault_*` variables, `releases.hashicorp.com/vault`, and Vault-specific install paths/binary names.
- **BREAKING**: Drop the `vault_type` (`community`/`enterprise`) variable and the `+ent` version-suffix behavior entirely. Terraform's CLI, unlike Vault, has no separate Enterprise binary distributed via `releases.hashicorp.com` — Terraform Enterprise is a self-hosted application, not a CLI download. There is exactly one Terraform binary per version/os/arch.
- Rename variables: `vault_version` → `terraform_version` (default bumped to `1.15.8`, the latest final Community release per `releases.hashicorp.com/terraform/index.json`), `vault_scope` → `terraform_scope`, `vault_mirror` → `terraform_mirror` (default `https://releases.hashicorp.com/terraform`). No replacement for `vault_type`.
- Update install/link paths and binary name from `vault` to `terraform` (e.g. `/usr/local/terraform/<version>/terraform`, symlinked at `/usr/local/bin/terraform`).
- Simplify `test.yml` from 4 scenarios (2 types × 2 scopes) down to 2 scenarios (system scope, local scope), since there is no type dimension anymore.
- Simplify `tests/verify.yml` to drop the Enterprise license-agreement (`LA_*`) and `notices` file assertions; Terraform's archive only ever bundles a single `LICENSE.txt` (BUSL text as of 1.5.5+), so only that assertion is kept.
- Update `meta/main.yml` galaxy info (`role_name: terraform`, `galaxy_tags: [terraform, hashicorp]`, description) and `README.md` (title, variable table, usage examples).
- Update `CLAUDE.md`'s "Upgrades" section to describe checking `https://releases.hashicorp.com/terraform/index.json` for Terraform releases instead of Vault's, and to drop the now-removed `vault_type` upgrade concern.
- No change to `meta/requirements.yml` (`andrewrothstein.unarchivedeps` is already pinned at the current latest, `3.2.0`) or `config.yaml` (CI base image tag `6.4.0` is already the latest published tag, and its `builds:` OS list already matches what that tag publishes) — verified against upstream sources during research; both are already current per `CLAUDE.md`'s upgrade checklist.

## Capabilities

### New Capabilities
- `terraform-install`: Ansible role capability that downloads a specific Terraform Community release archive, verifies it against HashiCorp's published SHA256 checksums, unarchives it into a version-namespaced install directory, and symlinks it into a bin directory — mirroring the existing Vault role's install/verify/symlink flow but without an Enterprise/Community type split.

### Modified Capabilities
(none — `openspec/specs/` has no pre-existing capability specs to modify; the Vault behavior was never captured in a spec, only in code)

## Impact

- Affected files: `tasks/main.yml`, `vars/main.yml`, `defaults/main.yml`, `meta/main.yml`, `README.md`, `test.yml`, `tests/verify.yml`, `handlers/main.yml` (comment only), `CLAUDE.md`.
- Unaffected: `meta/requirements.yml`, `config.yaml`, `.github/workflows/*` (none reference Vault or role-specific naming).
- No consumers depend on this role yet outside this repo, so the variable rename is a breaking change in name only — there is no external playbook to migrate.
