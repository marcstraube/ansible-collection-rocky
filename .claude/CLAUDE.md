# Claude Code — marcstraube.rocky Collection

## Overview

Rocky Linux installer collection. Generates Kickstart files for netinstall and
performs post-install configuration on deployed servers.

## Coding Standards (ansible-core 2.17+)

- **Facts:** `ansible_facts['os_family']` — never `ansible_os_family` (deprecated)
- **Modules:** Always FQCN (`ansible.builtin.*`, `community.general.*`)
- **Roles:** `ansible.builtin.include_role` — never `import_role`
- **Tasks:** `ansible.builtin.include_tasks` — never `import_tasks` for conditional dispatch
- **Tag propagation:** Always `apply: tags:` when using `include_tasks` inside blocks
- **Variables:** `<role>_<setting>`, internal: `__<role>_<setting>` (double underscore)
- **Booleans:** `<role>_enabled | default(true/false) | bool`
- **Templates:** `{{ ansible_managed | comment }}` header
- **Comments:** English only

### YAML Quoting Rules for Facts

| Context | Syntax | Example |
| --------- | -------- | --------- |
| `when:` clause (bare Jinja2) | Single quotes inside | `ansible_facts['os_family']` |
| Single-quoted YAML string | Double quotes inside | `'{{ ansible_facts["os_family"] }}'` |
| Double-quoted YAML string | Single quotes inside | `"{{ ansible_facts['os_family'] }}"` |
| Block scalar (`>`, `>-`, `\|`) | Single quotes inside | `ansible_facts['os_family']` |

## Role Conventions

### Directory Structure

```text
roles/<role_name>/
├── defaults/main.yml       # All variables with sensible defaults
├── handlers/main.yml       # Service restart/reload handlers
├── meta/main.yml           # Galaxy metadata, dependencies
├── tasks/
│   ├── main.yml            # Dispatcher: load OS vars, include OS tasks
│   ├── install.yml         # Installation tasks
│   ├── configure.yml       # Configuration tasks
│   ├── service.yml         # Service management
│   ├── btrfs.yml           # BTRFS optimizations (if applicable)
│   ├── firewall.yml        # Firewalld integration (if applicable)
│   └── apparmor.yml        # AppArmor integration (if applicable)
├── templates/*.j2          # Jinja2 templates
├── vars/
│   ├── Archlinux.yml       # OS-specific package names, paths, services
│   ├── Debian.yml
│   └── RedHat.yml          # Always latest (Rocky 10), overrides for older
└── molecule/default/       # Tests
    ├── molecule.yml
    ├── converge.yml
    └── verify.yml
```

### Task Patterns

- `main.yml` loads OS-specific vars with `include_vars` + `with_first_found`
- Use `include_tasks` (not `import_tasks`) for conditional OS dispatch
- `import_tasks` only for unconditional static includes in main.yml
- Always add `apply: tags:` when using `include_tasks` inside a block
- Use `ansible.builtin.` FQCN for all modules

### Templates

- Always include `{{ ansible_managed | comment }}` at the top
- Blank lines and descriptive comments go INSIDE `{% if %}` blocks, never before them
- No static blank line between `{% endif %}` and next `{% if %}`

### Examples in defaults/main.yml

- Always use generic placeholder data (`johndoe`, `john@example.com`)
- Never use real usernames or email addresses

## Roles

| Role        | Purpose                                                         |
|-------------|-----------------------------------------------------------------|
| `installer` | Rocky Linux installation - two modes via `rocky_installer_mode` |

### Installer Modes

- **`kickstart`** — Generates a Kickstart `.cfg` file locally (no remote connection needed)
- **`post_install`** — Configures a freshly installed Rocky server (runs on target host)

Dispatched in `tasks/main.yml` via `include_tasks` with `apply: tags:` and `when: rocky_installer_mode == '...'`.

## Variable Naming

- All variables use the `rocky_` prefix (not `installer_`): `rocky_version`, `rocky_disk`, `rocky_proxy`
- Internal variables: `__rocky_*` (double underscore prefix)
- Vault references: `rocky_root_password: "{{ vault_rocky_root_password }}"`

## Playbooks

| Playbook                           | Mode         | Connection   | Hosts           |
|------------------------------------|--------------|--------------|-----------------|
| `playbooks/generate-kickstart.yml` | kickstart    | `local`      | `rocky_servers` |
| `playbooks/post-install.yml`       | post_install | SSH (remote) | `rocky_servers` |

The generate-kickstart playbook runs entirely locally — it only generates files.
Post-install runs on the target and delegates to common roles (`users`, `sudo`, `openssh`, `package_management`).

## Key Rules

- **Rocky 10 requires x86-64-v3 (AVX2)** — does NOT boot on older CPUs (Westmere, Sandy Bridge).
  For old hardware, use `rocky_version: 9` in host_vars.
- **Kickstart output path:** Uses `{{ playbook_dir }}/.tmp/` — not `/tmp/` (sandbox restrictions)
- **Proxy support:** `rocky_proxy` applies to both kickstart mirror URLs and DNF configuration
- **SELinux:** Always `enforcing` by default — AppArmor is not used on Rocky/RHEL

## Testing

- Molecule with Podman, Rocky 10 image (`geerlingguy/docker-rockylinux10-ansible`)
- systemd in container: privileged mode, cgroup bind-mount

## Security

- Vault files (`vault.yml`) are never accessed — only `vault.yml.example` templates
- Use `{{ vault_* }}` variable references, never hardcode secrets
- `rocky_root_password` and `rocky_admin_ssh_key` must come from vault
- See root `.claude/CLAUDE.md` for vault protection rules
