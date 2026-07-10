# Terraform Ansible Role

Installs HashiCorp Terraform as a versioned binary with a symlink. See `README.md` for variables and usage.

## Upgrades

Two independent things can go stale and should be checked when revisiting this role:

1. **Terraform version default** (`defaults/main.yml`'s `terraform_version`): this role defaults to the latest final (non-`alpha`/`beta`/`rc`) Terraform release. Check `https://releases.hashicorp.com/terraform/index.json` for a newer final version (a version matching `\d+\.\d+\.\d+` with no `alpha`/`beta`/`rc` in the string) and bump `terraform_version` if one exists. Keep `test.yml`'s pinned `verify_version`/`terraform_version` values in sync with this default.
2. **`andrewrothstein.unarchivedeps` role dependency** (`meta/requirements.yml`): check its current latest release/tag and bump the pinned `version` if newer.
3. **CI base image** (`config.yaml`'s `upstream.tag`, `ghcr.io/bradfordwagner/ansible`): check `ghcr.io/v2/bradfordwagner/ansible/tags/list` (anonymous token from `ghcr.io/token?service=ghcr.io&scope=repository:bradfordwagner/ansible:pull`) for a newer semver tag. The `builds:` OS list must be updated to match whatever OS variants that new tag actually publishes — older OS families get dropped between releases (e.g. `debian_bookworm`, `rockylinux_9`, `ubuntu_jammy`, and `alpine_3.18/19/20` all disappeared by `6.4.0`), so bumping the tag alone without updating `builds:` breaks CI.

Ansible role dependencies and the CI base image are part of "upgrades" for this repo, not just the software the role installs.
