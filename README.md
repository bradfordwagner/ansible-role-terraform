# Terraform

Installs HashiCorp Terraform by downloading the official `.zip` release from `releases.hashicorp.com`, verifying it against the checksums published for that release, and unarchiving it into a versioned directory. A symlink is created in a bin directory pointing at the versioned binary.

This role is a binary installer only — it does not manage Terraform Cloud/Enterprise integration, workspaces, or state.

## Variables

| Variable | Default | Description |
|---|---|---|
| `terraform_version` | `1.15.8` | Terraform version to install. Check `https://releases.hashicorp.com/terraform/index.json` for newer releases. |
| `terraform_scope` | `system` | `system` or `local`. Drives the install directory, link directory, and `become` together — `system` installs under `/usr/local` with `become: true`; `local` installs under `{{ ansible_env.HOME }}/.local` with no privilege escalation. Not independently overridable; if you need a different combination, `terraform_scope` is the supported way to get it. |
| `terraform_mirror` | `https://releases.hashicorp.com/terraform` | Base URL to download releases and checksums from. |

## Usage

### System-wide install (defaults)

```yaml
- hosts: all
  roles:
    - role: bradfordwagner.terraform
```

Installs the pinned default version to `/usr/local/terraform/<version>/terraform`, symlinked at `/usr/local/bin/terraform`, using `become`.

### Local, user-scoped install (no become required)

```yaml
- hosts: all
  roles:
    - role: bradfordwagner.terraform
      terraform_scope: local
```

Installs to `{{ ansible_env.HOME }}/.local/terraform/<version>/terraform`, symlinked at `{{ ansible_env.HOME }}/.local/bin/terraform`, with no privilege escalation.

## Upgrades

See `CLAUDE.md` for the procedure to check for newer Terraform releases and newer versions of the `andrewrothstein.unarchivedeps` role dependency.
