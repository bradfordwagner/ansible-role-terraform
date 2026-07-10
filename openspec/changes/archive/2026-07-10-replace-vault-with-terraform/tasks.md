## 1. Core role variables

- [x] 1.1 In `defaults/main.yml`, replace `vault_version`/`vault_type`/`vault_scope`/`vault_mirror` with `terraform_version: '1.15.8'`, `terraform_scope: system`, `terraform_mirror: https://releases.hashicorp.com/terraform` (no `terraform_type` replacement — drop the type concept entirely)
- [x] 1.2 In `vars/main.yml`, rewrite all `vault_*` computed vars as `terraform_*` equivalents, removing the `_full_version`/`+ent` computation (use `terraform_version` directly wherever `vault_full_version` was used)

## 2. Task implementation

- [x] 2.1 In `tasks/main.yml`, rename all `vault_*` variable references to `terraform_*` and update task names/messages accordingly (dependency on `andrewrothstein.unarchivedeps` stays unchanged)

## 3. Handlers and metadata

- [x] 3.1 Update the comment in `handlers/main.yml` from "vault" to "terraform"
- [x] 3.2 Update `meta/main.yml`: `role_name: terraform`, description, and `galaxy_tags: [terraform, hashicorp]`

## 4. Tests

- [x] 4.1 Rewrite `test.yml` to two scenarios (system scope, local scope) using `terraform_version: '1.15.8'`, dropping the enterprise/community dimension
- [x] 4.2 Rewrite `tests/verify.yml` to re-derive `terraform_*` paths, drop the `verify_type`/enterprise branches (`LA_*` files, `notices` file, `verify_full_version` "+ent" computation), and keep the single `LICENSE.txt` assertion unconditional

## 5. Documentation

- [x] 5.1 Rewrite `README.md`: title, description, variable table (`terraform_version`, `terraform_scope`, `terraform_mirror`), and usage examples (system-wide and local-scope; drop the Enterprise example)
- [x] 5.2 Update `CLAUDE.md`'s "Upgrades" section: point `vault_version`/`releases.hashicorp.com/vault` references at `terraform_version`/`releases.hashicorp.com/terraform`, and remove the now-inapplicable Enterprise/`+ent` guidance

## 6. Verification

- [x] 6.1 Run the role's test playbook (`test.yml`) locally and confirm both scenarios pass, installing `terraform` and verifying the symlink and `LICENSE.txt`
