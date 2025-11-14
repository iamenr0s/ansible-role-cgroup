[![Molecule](https://github.com/iamenr0s/ansible-role-cgroup/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-cgroup/actions/workflows/molecule.yml) [![Release](https://github.com/iamenr0s/ansible-role-cgroup/actions/workflows/release.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-cgroup/actions/workflows/release.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/ansible_role_cgroup) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-cgroup/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-cgroup)

# Ansible Role: cgroup management

Manages Linux cgroup configuration across Debian/Ubuntu and RedHat/AlmaLinux/Rocky/Fedora. Ensures persistent kernel parameters via GRUB updates and reboots when changes are applied.

## Features

- Enforces cgroup v2 or v1 consistently
- Adds required kernel parameters idempotently
- Regenerates GRUB on supported distros
- Reboots the server when changes require it

## Requirements

- Ansible 2.9 or higher
- GRUB utilities available on target:
  - Debian/Ubuntu: `update-grub` (provided by `grub2-common`)
  - RedHat/Rocky/Alma/Fedora: `grub2-mkconfig` (provided by `grub2-tools`)

## Supported Platforms

- Debian: `bullseye`, `bookworm`, `trixie`
- Ubuntu: `focal`, `jammy`, `kinetic`
- EL: `8`, `9`, `10`
- Rocky: `8.0`, `9.0`, `10.0`
- Fedora: `39`–`42`

## Role Variables

- `cgroup_enforce_mode`: cgroup mode to enforce (`"v2"` or `"v1"`). Default: `"v2"`
- `cgroup_v1_enable_memory`: when `v1`, enables memory accounting with `cgroup_enable=memory swapaccount=1`. Default: `true`
- `cgroup_extra_kernel_params`: list of extra kernel `key=value` parameters to append. Default: `[]`

## Behavior

- v2 mode adds `systemd.unified_cgroup_hierarchy=1` and `cgroup_no_v1=all`
- v1 mode adds `systemd.unified_cgroup_hierarchy=0`; optionally memory accounting params
- Updates both `GRUB_CMDLINE_LINUX` and `GRUB_CMDLINE_LINUX_DEFAULT`
- Regenerates GRUB and reboots only when changes were applied

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

## Testing

Molecule scenario uses Podman containers. `prepare.yml` installs GRUB tools so that `update-grub`/`grub2-mkconfig` succeed inside containers. Converge asserts kernel parameters are present in `/etc/default/grub`.

Run:

```bash
molecule test
```

## License

MIT

## Author Information

Author: iamenr0s
Galaxy: `iamenr0s.ansible_role_cgroup`

## Contributing

Contributions are welcome! Please:
- Fork the repository
- Create a feature branch
- Make changes and add tests
- Submit a pull request

## Changelog

See `CHANGELOG.md` for version history and release notes.
