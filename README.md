[![Molecule](https://github.com/iamenr0s/ansible-role-kernel-configuration/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-kernel-configuration/actions/workflows/molecule.yml) [![Release](https://github.com/iamenr0s/ansible-role-kernel-configuration/actions/workflows/release.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-kernel-configuration/actions/workflows/release.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/ansible_role_upgrade) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-kernel-configuration/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-kernel-configuration)

# Ansible Role: Kernel Parameter Configuration

This role helps keep your Linux servers healthy and consistent by applying sensible kernel settings tailored to each supported OS version. It reduces guesswork, improves stability and performance, and makes changes easy to roll out and keep across reboots. Use it as a reliable baseline you can tweak to fit your environment.

## Features

- Applies kernel parameters using `sysctl` based on distro major version
- Persists settings in `{{ kernel_parameters_sysctl_file }}` and optionally reloads
- Clean defaults with per-platform overrides; easy to customize

## Requirements

- `ansible` >= 2.9
- Collections: `ansible.posix`, `community.general`

## Supported Platforms

- Ubuntu 20.04, 22.04, 24.04
- Debian 11, 12
- RHEL 7, 8, 9
- Rocky Linux 8, 9, 10
- AlmaLinux 8, 9, 10
- Fedora 39+

## Role Variables

### Basic Configuration

- `kernel_parameters_sysctl_file`: Path to persistent sysctl file (default: `/etc/sysctl.d/99-kernel-parameters.conf`)
- `kernel_parameters_reload`: Reload sysctl after changes (default: `true`)
- `kernel_parameters_common`: Dict of parameters applied to all platforms
- `kernel_parameters_major_map`: Nested dict mapping `distribution -> major_version -> {param: value}`

Example:

```yaml
kernel_parameters_common:
  net.ipv4.tcp_syncookies: 1
  net.ipv4.ip_forward: 0

kernel_parameters_major_map:
  Ubuntu:
    "22":
      vm.swappiness: 10
      net.core.somaxconn: 2048
  RedHat:
    "8":
      fs.file-max: 200000
```

## Dependencies

None.

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: iamenr0s.ansible_role_kernel_configuration
      vars:
        kernel_parameters_common:
          net.ipv4.tcp_syncookies: 1
        kernel_parameters_major_map:
          Debian:
            "12":
              vm.swappiness: 5
```

## Testing

- Use Molecule with Docker or Vagrant to validate parameter application across platforms.

## Troubleshooting

- Ensure `ansible_distribution` and `ansible_distribution_major_version` are detected correctly.
- If parameters don’t persist, verify `kernel_parameters_sysctl_file` exists and is included by your system’s sysctl configuration.

## License

MIT

## Author Information

Author: iamenr0s
Galaxy: `iamenr0s.ansible_role_kernel_configuration`

## Contributing

Contributions are welcome! Please:
- Fork the repository
- Create a feature branch
- Make your changes
- Add tests if applicable
- Submit a pull request

## Changelog

See `CHANGELOG.md` for version history and release notes.
