[![Molecule](https://github.com/iamenr0s/ansible-role-kernel-configuration/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-kernel-configuration/actions/workflows/molecule.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/ansible_role_kernel_configuration) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-kernel-configuration/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-kernel-configuration)

Ansible Role: Kernel Parameter Configuration
=============================================

This role applies sensible kernel settings via `sysctl` tailored to each supported OS major version, and manages kernel modules (load, blacklist, options). It's container-aware: sysctl work is skipped automatically when the role detects it's running inside a container, so the same playbook is safe to run against both real hosts and containerized test/CI environments.

Features
--------
- Applies kernel parameters using `sysctl`, keyed by distro and major version.
- Persists settings in `kernel_parameters_sysctl_file` and optionally reloads.
- Skips sysctl work automatically inside containers (Docker/Podman) — no manual guard needed in your playbook.
- Manages kernel modules: load/unload, persistent loading, blacklisting, and per-module options.

Requirements
------------
- Python 3 available on the managed hosts (Ansible modules require Python).
- Collections: `ansible.posix`, `community.general` (see `requirements.yml`).
- Run with privilege escalation on real hosts: `become: true` is recommended.

Supported Platforms
--------------------
- AlmaLinux 8, 9, 10
- Debian 12, 13
- Fedora 42, 43, 44
- Rocky Linux 8, 9, 10
- Ubuntu 22.04, 24.04

Role Variables
---------------
Defined in `defaults/main.yml` (mirrored in `meta/argument_specs.yml`):

### Sysctl / kernel parameters

- `kernel_parameters_sysctl_file` (str): Path to the persistent sysctl file (default: `/etc/sysctl.d/99-kernel-parameters.conf`).
- `kernel_parameters_reload` (bool): Reload sysctl after applying changes (default: `true`).
- `kernel_parameters_reload_in_containers` (bool): Reload behavior used when the host is detected as a container (default: `false`).
- `kernel_parameters_sysctl_ignoreerrors` (bool): Ignore errors when applying sysctl parameters — use with caution (default: `false`).
- `kernel_env_path` (str): `PATH` used when running sysctl commands (default: `/usr/sbin:/sbin:{{ ansible_env.PATH | default('') }}`).
- `kernel_parameters_common` (dict): Parameters applied to all platforms unless overridden.
- `kernel_parameters_major_map` (dict): Nested dict mapping `ansible_distribution -> ansible_distribution_major_version -> {param: value}`. A distro/version missing from this map still works — the role falls back to `{}` for that host.

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

### Kernel modules

- `kernel_modules_load` (list): Module names to load immediately and persist (default: `[]`).
- `kernel_modules_blacklist` (list): Module names to blacklist, preventing them from loading (default: `[]`).
- `kernel_modules_options` (dict): Map of module name to an options dict applied when loading.
- `kernel_modules_persistent` (bool): Persist module load/options across reboots (default: `true`).
- `kernel_modules_load_conf_dir` (str): Directory used for persisting immediate module loads (default: `/etc/modules-load.d`).
- `kernel_modules_modprobe_conf_dir` (str): Directory used for persisting module options/blacklists (default: `/etc/modprobe.d`).
- `kernel_modules_blacklist_file` (str): File used to persist the module blacklist (default: `/etc/modprobe.d/blacklist-ansible-role-kernel.conf`).
- `kernel_modules_options_file` (str): File used to persist standalone module options (default: `/etc/modprobe.d/ansible-role-kernel-options.conf`).

Example:

```yaml
kernel_modules_load:
  - 8021q
  - dummy
kernel_modules_options:
  dummy:
    numdummies: 2
kernel_modules_blacklist:
  - nouveau
kernel_modules_persistent: true
```

Notes:
- Options are applied when modules are loaded by this role.
- Blacklisted modules are also unloaded if currently loaded.

Container Behavior
-------------------
The role detects containers via `ansible_virtualization_type`/`ansible_virtualization_role` and sets `kernel_in_container`. When true:
- The sysctl directory is not created and no kernel parameters are applied (they usually can't be, and doing so can trigger a PAM/sudo failure under `become`).
- Module loading (`modprobe`) errors are ignored rather than failing the play.
- Module blacklisting still runs — it's just writing a config file, safe in any environment.

Example Playbook
-----------------
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
        kernel_modules_load:
          - 8021q
          - dummy
        kernel_modules_options:
          dummy:
            numdummies: 2
        kernel_modules_blacklist:
          - nouveau
```

Contributing & Security
-------------------------
- Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
- Report vulnerabilities privately per [SECURITY.md](SECURITY.md); do not open public issues for them.

CI & Release (maintainers)
----------------------------
A single workflow (`.github/workflows/molecule.yml`) runs lint and the full Molecule distro matrix on pushes to `main`, PRs, and `v*` tags. On `v*` tags, a `release` job publishes to Ansible Galaxy after all tests pass.

The Galaxy API key lives in the `galaxy` GitHub environment, which only `v*` tags may target. One-time setup:

```bash
# Galaxy publishing key (environment-scoped, get it from galaxy.ansible.com/ui/token)
gh secret set GALAXY_API_KEY --env galaxy --repo iamenr0s/ansible-role-kernel-configuration

# Code scanning notifications (Slack webhook URL; for Discord append /slack to the webhook URL)
gh secret set SECURITY_ALERT_WEBHOOK --env galaxy --repo iamenr0s/ansible-role-kernel-configuration
```

`.github/workflows/code-scanning-notify.yml` polls the code-scanning API every 6 hours and posts new or updated open alerts to that webhook (GitHub Actions cannot trigger on `code_scanning_alert` directly).

To release: tag a commit `vX.Y.Z` and push the tag — CI gates the Galaxy publish.

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the
local pipeline commands and pull request checklist. This project follows the
[Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).

## Security

See [SECURITY.md](SECURITY.md) — GitHub private vulnerability reporting, no
public issues for security bugs.

## License

This project is licensed under the [MIT License](LICENSE).

## Author Information

Author: iamenr0s

Galaxy: `iamenr0s.ansible_role_kernel_configuration`
