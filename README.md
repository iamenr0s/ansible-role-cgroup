[![Molecule](https://github.com/iamenr0s/ansible-role-cgroup/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-cgroup/actions/workflows/molecule.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/ansible_role_cgroup) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-cgroup/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-cgroup)

# Ansible Role: cgroup management

Manages Linux cgroup configuration across Debian/Ubuntu and RHEL-family (AlmaLinux/RockyLinux/Fedora) servers. Ensures persistent kernel parameters via GRUB updates and reboots when changes are applied.

## Features

- Enforces cgroup v2 or v1 consistently
- Adds required kernel parameters idempotently
- Regenerates GRUB on supported distros
- Skips GRUB regeneration/reboot automatically when running inside a container
- Reboots the server when changes require it (and when the memory cgroup controller is expected but absent)

## Requirements

- Ansible 2.13 or higher (uses `lineinfile`'s `found` return attribute)
- GRUB utilities available on target:
  - Debian/Ubuntu: `update-grub` (provided by `grub2-common`)
  - RedHat/Rocky/Alma/Fedora: `grub2-mkconfig` (provided by `grub2-tools`)

## Supported Platforms

- AlmaLinux 8, 9, 10
- Debian 12, 13
- Fedora 42, 43, 44
- Rocky 8, 9, 10 (covered under EL in `meta/main.yml`)
- Ubuntu 22.04, 24.04

## Role Variables

Defined in `defaults/main.yml`:

- `cgroup_enforce_mode` (str): cgroup mode to enforce, `"v2"` or `"v1"`. Default: `"v2"`
- `cgroup_v1_enable_memory` (bool): when `v1`, enables memory accounting with `cgroup_enable=memory swapaccount=1`. Default: `true`
- `cgroup_v1_add_cgroup_memory_param` (bool): when `v1` and `cgroup_v1_enable_memory`, also adds `cgroup_memory=1` for kernels that require an explicit enable. Default: `true`
- `cgroup_force_reboot_on_memcg_absent` (bool): force a reboot when the memory cgroup controller is expected (v2, or v1 with `cgroup_v1_enable_memory`) but not detected on the running kernel. Default: `true`
- `cgroup_extra_kernel_params` (list): extra kernel `key=value` parameters to append. Default: `[]`
- `cgroup_allow_reboot` (bool): set to `false` to disable automatic reboots entirely, e.g. for environments where reboots must be scheduled manually. Default: `true`

## Behavior

- v2 mode adds `systemd.unified_cgroup_hierarchy=1` and `cgroup_no_v1=all`
- v1 mode adds `systemd.unified_cgroup_hierarchy=0`; optionally memory accounting params
- Updates both `GRUB_CMDLINE_LINUX` and `GRUB_CMDLINE_LINUX_DEFAULT`
- Regenerates GRUB and reboots only when changes were applied, and skips both entirely when a container is detected (Docker/Podman/LXC/LXD, overlay rootfs, or container marker files)
- Checks whether the memory cgroup controller is actually active on the running kernel; reboots via `cgroup_force_reboot_on_memcg_absent` if it's expected but absent

## Example Playbook

Basic usage (v2 enforced):

```yaml
- hosts: all
  become: true
  roles:
    - role: iamenr0s.ansible_role_cgroup
```

Enforce v1 with memory accounting and extra params:

```yaml
- hosts: all
  become: true
  vars:
    cgroup_enforce_mode: "v1"
    cgroup_v1_enable_memory: true
    cgroup_extra_kernel_params:
      - audit=1
  roles:
    - role: iamenr0s.ansible_role_cgroup
```

## Notes

- Set `cgroup_allow_reboot: false` if reboots must be scheduled manually rather than happening automatically when configuration changes.
- Inside containers (Docker/Podman/LXC/LXD), GRUB regeneration and reboot are automatically skipped — the role detects this via `ansible_virtualization_type`, overlay rootfs, and container marker files.

## Contributing & Security

- Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
- Report vulnerabilities privately per [SECURITY.md](SECURITY.md); do not open public issues for them.

## CI & Release (maintainers)

A single workflow (`.github/workflows/molecule.yml`) runs lint and the full Molecule distro matrix on pushes to `main`, PRs, and `v*` tags. On `v*` tags, a `release` job publishes to Ansible Galaxy after all tests pass.

The Galaxy API key lives in the `galaxy` GitHub environment, which only `v*` tags may target. One-time setup:

```bash
# Galaxy publishing key (environment-scoped, get it from galaxy.ansible.com/ui/token)
gh secret set GALAXY_API_KEY --env galaxy --repo iamenr0s/ansible-role-cgroup

# Code scanning notifications (Slack webhook URL; for Discord append /slack to the webhook URL)
gh secret set SECURITY_ALERT_WEBHOOK --env galaxy --repo iamenr0s/ansible-role-cgroup
```

`.github/workflows/code-scanning-notify.yml` polls the code-scanning API every 6 hours and posts new or updated open alerts to that webhook (GitHub Actions cannot trigger on `code_scanning_alert` directly).

To release: tag a commit `vX.Y.Z` and push the tag — CI gates the Galaxy publish. See `CHANGELOG.md` for version history and release notes.

## License

MIT

## Author Information

Author: iamenr0s
Galaxy: `iamenr0s.ansible_role_cgroup`
