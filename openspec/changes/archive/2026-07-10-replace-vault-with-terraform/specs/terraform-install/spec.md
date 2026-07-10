## ADDED Requirements

### Requirement: Versioned Terraform binary install
The role SHALL download the Terraform Community release archive for a configurable `terraform_version` from a configurable `terraform_mirror`, verify it against the SHA256 checksum published alongside that release, and unarchive it into an install directory namespaced by `terraform_version` (e.g. `<parent>/terraform/<version>/`), skipping re-download/re-verification only when the binary for that exact version is already present. Because the install directory embeds the version, changing `terraform_version` always targets a new, not-yet-installed path — the skip never suppresses an upgrade, it only avoids redundant re-fetching of a version already installed.

#### Scenario: Fresh install
- **WHEN** the role runs with `terraform_version: '1.15.8'` and no prior install exists at the computed install directory
- **THEN** the role downloads `terraform_<version>_<os>_<arch>.zip` from `{{ terraform_mirror }}/{{ terraform_version }}/`, verifies it against the matching entry in `terraform_<version>_SHA256SUMS`, and unarchives it so `terraform` executable exists at `<install_dir>/terraform`

#### Scenario: Already installed at the same version
- **WHEN** the role runs with a `terraform_version` whose `<install_dir>/terraform` already exists
- **THEN** the role SHALL NOT re-download or re-verify the archive for that version

#### Scenario: Version bump triggers a new install
- **WHEN** the role runs with a `terraform_version` different from a previously-installed version (a different `<install_dir>`)
- **THEN** the role downloads, verifies, and installs the new version at its own install directory, leaving the previous version's install directory untouched, and updates the bin symlink to point at the new version

### Requirement: Bin directory symlink
The role SHALL create (or update) a symlink in a configurable bin directory pointing at the versioned Terraform executable, so a stable path (e.g. `terraform`) always resolves to the installed version.

#### Scenario: Symlink created or updated
- **WHEN** the role completes a run for a given `terraform_version`
- **THEN** `<link_dir>/terraform` is a symlink pointing at `<install_dir>/terraform`, replacing any prior symlink target

### Requirement: System and local install scopes
The role SHALL support a `terraform_scope` of `system` (installs under `/usr/local`, uses `become` for all privileged file operations) or `local` (installs under `{{ ansible_env.HOME }}/.local`, no privilege escalation), driving install directory, link directory, and privilege escalation together as a single choice.

#### Scenario: System scope
- **WHEN** `terraform_scope` is `system` (the default)
- **THEN** the role installs to `/usr/local/terraform/<version>/terraform`, symlinks at `/usr/local/bin/terraform`, and uses `become: true` for file operations

#### Scenario: Local scope
- **WHEN** `terraform_scope` is `local`
- **THEN** the role installs to `{{ ansible_env.HOME }}/.local/terraform/<version>/terraform`, symlinks at `{{ ansible_env.HOME }}/.local/bin/terraform`, and performs no privilege escalation

### Requirement: Bundled license file preserved
The role SHALL leave the `LICENSE.txt` file that HashiCorp bundles inside the Terraform release archive in place alongside the installed binary, without modification.

#### Scenario: License file present after install
- **WHEN** a Terraform release is installed
- **THEN** `<install_dir>/LICENSE.txt` exists as extracted from the release archive
