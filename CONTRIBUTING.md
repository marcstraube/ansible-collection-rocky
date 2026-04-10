# Contributing to marcstraube.rocky

Thank you for your interest in contributing! This document provides guidelines
and instructions for contributing to this collection.

## Development Setup

### Prerequisites

- Python 3.13+
- ansible-core >= 2.17
- Podman (for Molecule tests)
- Git
- pre-commit

### Clone and Setup

```bash
git clone https://github.com/marcstraube/ansible-collection-rocky.git
cd ansible-collection-rocky

pip install ansible-core ansible-lint molecule molecule-plugins[podman] yamllint pre-commit

ansible-galaxy collection install marcstraube.common community.general community.crypto ansible.posix

pre-commit install
```

## Pre-commit Hooks

```bash
# Run all hooks
pre-commit run --all-files

# Run specific hook
pre-commit run ansible-lint --all-files
pre-commit run yamllint --all-files
```

### Configured Hooks

- **ansible-lint** - Ansible best practices and syntax
- **yamllint** - YAML syntax and formatting
- **markdownlint** - Markdown formatting
- **trailing-whitespace** - Remove trailing whitespace
- **end-of-file-fixer** - Ensure files end with newline
- **check-yaml** - Validate YAML syntax
- **detect-secrets** - Prevent accidental secret commits

## Running Tests

### Linting

```bash
ansible-lint
yamllint -c .yamllint .
```

### Molecule Tests

```bash
# Test the installer role
cd roles/installer
molecule test

# Step by step (for debugging)
molecule create
molecule converge
molecule verify
molecule destroy

# Login to container
molecule login
```

## Code Style

### Ansible Conventions

- FQCN for all modules: `ansible.builtin.*`, `community.general.*`
- Facts syntax: `ansible_facts['key']` (never `ansible_key`)
- `include_tasks` for conditional dispatch, `import_tasks` for static includes
- Tag propagation: `apply: tags:` on `include_tasks`
- Variables: `<role>_<setting>`, internal: `__<role>_<setting>`
- Boolean toggles: `<role>_enabled | default(true) | bool`
- Comments: English only

### Variable Naming

```yaml
# Role variables (defaults/main.yml)
rocky_version: 10
rocky_disk:
  device: sda

# Internal/OS-specific (vars/*.yml)
__rocky_root_password_hash: "..."
```

### Examples in defaults/main.yml

Always use generic placeholder data in commented examples:

```yaml
# Good
#   hostname: rocky-server
#   ip: "192.168.1.100"

# Bad — never use real hostnames/IPs
#   hostname: prod-db-01
```

## Pull Request Process

1. Run pre-commit hooks: `pre-commit run --all-files`
2. Test changes: `molecule test`
3. Update documentation if needed
4. Use conventional commit format:
   - `feat(installer): add BTRFS partitioning support`
   - `fix(installer): correct LVM volume sizing`
   - `docs: update installation instructions`

## License

By contributing, you agree that your contributions will be licensed under
the MIT License.
