# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Install dependencies
```bash
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
```

### Lint
```bash
yamllint .
ansible-lint
```

### Run Molecule tests (requires Podman)
```bash
# Default distro (almalinux10)
molecule test

# Specific distro
MOLECULE_DISTRO=debian12 molecule test

# Only converge (no destroy)
MOLECULE_DISTRO=ubuntu2204 molecule converge

# Run with verbose output
molecule -vvv test
```

### Supported `MOLECULE_DISTRO` values
`almalinux8`, `almalinux9`, `almalinux10`, `debian11`, `debian12`, `fedora40`, `fedora41`, `fedora42`, `rockylinux8`, `rockylinux9`, `rockylinux10`, `ubuntu2204`, `ubuntu2404`

CI uses Docker driver; local testing uses Podman driver (configured in `molecule/default/molecule.yml`).

## Architecture

### Role purpose
Manages Linux cgroup configuration by modifying GRUB kernel command-line parameters and rebooting when changes are applied. Supports both cgroup v1 and v2 across Debian/Ubuntu and RedHat/EL family distros.

### Task flow (`tasks/main.yml`)
1. **Build kernel args list** (`__cgroup_kernel_args`) — computed from role variables; v2 sets `systemd.unified_cgroup_hierarchy=1 cgroup_no_v1=all`, v1 adds memory controller params conditionally.
2. **GRUB file management** — ensures `/etc/default/grub` exists, both `GRUB_CMDLINE_LINUX` and `GRUB_CMDLINE_LINUX_DEFAULT` keys are present.
3. **Option patching** — corrects mismatched values already in cmdline, then appends any missing options.
4. **Reboot flag accumulation** — `cgroup_reboot_required` is set to `true` if any of the above steps changed anything.
5. **Container detection** — sets `cgroup_skip_grub_update` to skip `update-grub`/`grub2-mkconfig` when running inside Docker/Podman/LXC (detected via `ansible_virtualization_type`, overlay rootfs, or marker files).
6. **GRUB regeneration** — runs `update-grub` (Debian) or `grub2-mkconfig -o <path>` (RedHat) only when `cgroup_reboot_required` and not in a container.
7. **Memory controller check** — detects whether the memory cgroup controller is actually active; triggers reboot via `cgroup_force_reboot_on_memcg_absent` if it is expected but absent.
8. **Reboot** — handled by the single handler in `handlers/main.yml`.

### Key role variables (`defaults/main.yml`)
| Variable | Default | Effect |
|---|---|---|
| `cgroup_enforce_mode` | `"v2"` | `"v1"` or `"v2"` |
| `cgroup_v1_enable_memory` | `true` | Adds `cgroup_enable=memory swapaccount=1` (v1 only) |
| `cgroup_v1_add_cgroup_memory_param` | `true` | Also adds `cgroup_memory=1` (v1 only) |
| `cgroup_force_reboot_on_memcg_absent` | `true` | Forces reboot when memory controller expected but not found |
| `cgroup_extra_kernel_params` | `[]` | Appended verbatim to the computed kernel arg list |
| `cgroup_allow_reboot` | `true` | Set to `false` to suppress all automatic reboots (e.g. for manually-scheduled maintenance windows) |

### Naming conventions
- Private/computed facts use the `__` prefix (e.g. `__cgroup_kernel_args`, `__grub_keys`).
- Task names follow the `ansible-lint` `name[prefix]` rule: `"{stem} | Description"` format is enforced by `task_name_prefix: "{stem} | "` in `.ansible-lint`.

### CI/CD
- **Molecule workflow** (`.github/workflows/molecule.yml`) — runs lint then a matrix of distros on every PR and on version tags (`v*.*.*`).
- **Release workflow** (`.github/workflows/release.yml`) — publishes to Ansible Galaxy after Molecule succeeds, triggered on version tags.
