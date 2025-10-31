# Changelog

All notable changes to this project are documented in this file.

## [1.0.0] - 2025-10-28
- Initial release of `ansible-role-kernel-configuration`.
- Adds kernel parameter management per distribution major version.
- Provides persistent sysctl configuration via `/etc/sysctl.d/99-kernel-parameters.conf`.
- Includes root-level role layout: `meta/`, `defaults/`, `tasks/`, `handlers/`.
- Documents usage and variables in `README.md` with a user-friendly overview.
- Adds example playbook under `examples/playbook.yml`.
- Mirrors supported platforms from `meta/main.yml` (Ubuntu, Debian, RHEL, Rocky, AlmaLinux, Fedora, Amazon Linux, SLES).

## [1.1.0] - 2025-10-31
- Add kernel modules management:
  - Load modules immediately and persist via system configuration
  - Blacklist modules and unload if currently loaded
  - Pass module options when loading
- Introduce new variables under `defaults/` and static paths under `vars/`
- Update documentation (README) and examples
- Ensure required collections are declared (`community.general`, `ansible.posix`)

## Format
This changelog follows a simple, readable format. Future versions may adopt the
Keep a Changelog style and semantic versioning.
