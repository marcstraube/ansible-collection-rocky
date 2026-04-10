# marcstraube.rocky.installer

Rocky Linux installation via Kickstart file generation and post-install bootstrap.

## Description

This role operates in two modes controlled by `rocky_installer_mode`:

- **`kickstart`** — Generates a Kickstart `.cfg` file locally for unattended
  Rocky Linux netinstall. No remote connection needed.
- **`post_install`** — Bootstraps a freshly installed Rocky Linux system
  (verifies OS, configures proxy, runs initial update). User management,
  SSH hardening, and sudo are handled by common roles via the post-install
  playbook.

## Requirements

- ansible-core >= 2.17
- `marcstraube.common` collection (for post-install playbook)

## Supported Platforms

| Platform    | Versions |
| ----------- | -------- |
| Rocky Linux | 9, 10    |

## Role Variables

### Role Control

| Variable               | Default     | Description                          |
| ---------------------- | ----------- | ------------------------------------ |
| `rocky_installer_mode` | `kickstart` | Mode: `kickstart` / `post_install`   |

### Rocky Linux Version

| Variable        | Default  | Description               |
| --------------- | -------- | ------------------------- |
| `rocky_version` | `10`     | Rocky Linux major version |
| `rocky_release` | `latest` | Release version           |

### Network Configuration

| Variable                  | Default | Description                                                                 |
| ------------------------- | ------- | --------------------------------------------------------------------------- |
| `rocky_kickstart_network` | (dict)  | Network config (method, device, hostname, ip, netmask, gateway, nameserver) |
| `rocky_proxy`             | `""`    | HTTP proxy for package downloads                                            |

### Disk / Partitioning

| Variable     | Default | Description                                     |
| ------------ | ------- | ----------------------------------------------- |
| `rocky_disk` | (dict)  | Disk config (device, method, vg_name, lv sizes) |

### Security

| Variable                            | Default     | Description                |
| ----------------------------------- | ----------- | -------------------------- |
| `rocky_selinux`                     | `enforcing` | SELinux mode               |
| `rocky_root_password`               | vault ref   | Root password (use vault!) |
| `rocky_kickstart_firewall_services` | `[ssh]`     | Firewall services to allow |

### Admin User

| Variable              | Default | Description              |
| --------------------- | ------- | ------------------------ |
| `rocky_admin_user`    | `admin` | Admin username           |
| `rocky_admin_ssh_key` | `""`    | SSH public key for admin |

### Output

| Variable                 | Default               | Description                          |
| ------------------------ | --------------------- | ------------------------------------ |
| `rocky_kickstart_output` | see defaults/main.yml | Generated kickstart file output path |

## Tags

| Tag                  | Scope                       |
| -------------------- | --------------------------- |
| `rocky`              | All installer tasks         |
| `rocky:kickstart`    | Kickstart file generation   |
| `rocky:post-install` | Post-install bootstrap      |

## Example Playbook

### Generate Kickstart File

```yaml
- name: Generate Rocky Linux Kickstart
  hosts: rocky_servers
  gather_facts: false
  connection: local

  tasks:
    - name: Include installer role
      ansible.builtin.include_role:
        name: marcstraube.rocky.installer
      vars:
        rocky_installer_mode: kickstart
```

### Post-Install Bootstrap

```yaml
- name: Rocky Linux Post-Install
  hosts: rocky_servers
  become: true

  tasks:
    - name: Include installer role
      ansible.builtin.include_role:
        name: marcstraube.rocky.installer
      vars:
        rocky_installer_mode: post_install
```

## Testing

```bash
cd roles/installer
molecule test
```

- Driver: Podman
- Platforms: Rocky Linux 9, Rocky Linux 10

## Notes

- Rocky 10 requires x86-64-v3 (AVX2). For older CPUs, use `rocky_version: 9`.
- The Kickstart `%post` section bootstraps the admin user and SSH key. The
  post-install playbook then delegates to common roles for ongoing management.

## License

MIT

## Author

Marc Straube (<email@marcstraube.de>)
