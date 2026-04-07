# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Fixed
- String boolean bug in `cgroup_reboot_required` accumulation — `set_fact` returned strings
  causing `"False"` to be truthy, potentially triggering unintended reboots
- `lineinfile.found` attribute now guarded with `| default(0)` for Ansible compatibility
- `grub2-mkconfig` command now uses `argv` list form to avoid shell interpretation issues
- `/bin/true` handler-trigger hack replaced with `ansible.builtin.debug`
- Misleading `flush_handlers` task name corrected to "Flush pending handlers"
- Molecule driver mismatch: switched from `podman` to `docker` to match CI environment
- CI vault password block removed — no vault-encrypted content exists in the repo
- `examples/playbook.yml` leading-space indentation corrected
- Stale Ubuntu `kinetic` (EOL) replaced with `noble` in `meta/main.yml`
- Molecule prepare stage failing on EL8 (AlmaLinux 8, Rocky Linux 8) — Ansible 2.20+
  generates `AnsiballZ_setup.py` using `from __future__ import annotations` (Python 3.7+)
  but EL8 defaults to Python 3.6; fixed by bootstrapping `python39` via `raw` before
  facts gathering, and using `raw` for `grub2-tools` install to avoid `python3-dnf`
  binding requirement

### Added
- `cgroup_allow_reboot` variable (default: `true`) to allow disabling automatic reboots
- `cgroup_no_v1=all` assertion added to Molecule converge playbook
- `molecule/default/verify.yml` — dedicated post-convergence verification playbook
- ansible-lint added to CI lint job
- `molecule` CI job now requires `lint` to pass first (`needs: lint`)

### Changed
- `min_ansible_version` bumped from `2.9` to `2.13` (required for `lineinfile.found`)
- CI Python upgraded from `3.9` (EOL) to `3.12`
- `actions/checkout` pinned to `@v5` in both workflows for consistency
- Unused `ansible.posix` and `community.general` collection dependencies removed
- `.ansible-lint` cleaned up — removed project-irrelevant boilerplate

## [0.1.0] - 2025-11-14

- Add cross-distro cgroup management (Debian/Ubuntu, EL/Rocky/Alma/Fedora)
- Persist kernel params via GRUB updates
- Conditional reboot when configuration changes
- Molecule scenario installs GRUB tools and asserts configuration
- Documentation updates (README)
