# Ansible Collection: marcstraube.rocky

[![CI](https://github.com/marcstraube/ansible-collection-rocky/workflows/CI/badge.svg)](https://github.com/marcstraube/ansible-collection-rocky/actions)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Sponsor](https://img.shields.io/badge/sponsor-PayPal-blue.svg)](https://paypal.me/marcstraube)

## Description

Automated Rocky Linux server installation via Kickstart netinstall with
post-install configuration. Generates Kickstart files for unattended installs
and prepares freshly installed systems for Ansible management.

## Supported Platforms

- Rocky Linux 9
- Rocky Linux 10

## Requirements

- ansible-core >= 2.17
- `marcstraube.common` collection (for post-install playbook)

## Included Roles

| Role          | Description                                            |
| ------------- | ------------------------------------------------------ |
| **installer** | Kickstart generation and post-install configuration    |

### Installer Modes

The installer role operates in two modes controlled by `rocky_installer_mode`:

| Mode           | Purpose                                    | Connection |
| -------------- | ------------------------------------------ | ---------- |
| `kickstart`    | Generate Kickstart `.cfg` file locally     | local      |
| `post_install` | Configure freshly installed Rocky server   | SSH        |

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install marcstraube.rocky
```

### From Git

```bash
ansible-galaxy collection install git+https://github.com/marcstraube/ansible-collection-rocky.git,main
```

### Requirements File

```yaml
# requirements.yml
collections:
  - name: marcstraube.rocky
    version: ">=1.0.0"
```

## Usage

### Generate Kickstart File

```yaml
- name: Generate Rocky Linux Kickstart
  hosts: rocky_servers
  gather_facts: false
  connection: local

  tasks:
    - name: Include installer role (kickstart mode)
      ansible.builtin.include_role:
        name: marcstraube.rocky.installer
      vars:
        rocky_installer_mode: kickstart
```

Or use the included playbook:

```bash
ansible-playbook collections/ansible_collections/marcstraube/rocky/playbooks/generate-kickstart.yml \
  -i inventories/example -l myserver
```

### Post-Install Configuration

```yaml
- name: Rocky Linux Post-Install
  hosts: rocky_servers
  become: true

  tasks:
    - name: Include installer role (post_install mode)
      ansible.builtin.include_role:
        name: marcstraube.rocky.installer
      vars:
        rocky_installer_mode: post_install
```

### Key Variables

```yaml
# Rocky version (9 or 10)
rocky_version: 10

# Network configuration
rocky_kickstart_network:
  method: static       # dhcp | static
  device: eth0
  hostname: rocky-server
  ip: "192.168.1.100"
  netmask: "255.255.255.0"
  gateway: "192.168.1.1"
  nameserver: "1.1.1.1"

# Disk partitioning
rocky_disk:
  device: sda
  method: lvm           # auto | lvm
  vg_name: rocky
  lv_root_size: 0       # 0 = remaining space
  lv_swap_size: 4096    # MB

# Security
rocky_selinux: enforcing
rocky_root_password: "{{ vault_rocky_root_password }}"

# Admin user
rocky_admin_user: admin
rocky_admin_ssh_key: "{{ vault_rocky_admin_ssh_key }}"
```

See `roles/installer/defaults/main.yml` for all available variables.

## Testing

The installer role is tested with Molecule using Podman containers.

```bash
cd roles/installer
molecule test
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and guidelines.

## License

MIT

## Author

Marc Straube (<email@marcstraube.de>)
