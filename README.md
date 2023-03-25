# Ansible Role: Teleport Node Service

[![Ansible Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-mdsketch.teleport-blueviolet)](https://galaxy.ansible.com/mdsketch/teleport)


An ansible role to install or update the teleport node service and teleport config on Debian based systems.

Works with any architecture that teleport has a binary for, see available [teleport downloads](https://goteleport.com/teleport/download/).

If you add your own teleport config file template you can run any node services you want (ssh, app, database, kubernetes)

## Requirements

A running teleport cluster so that you can provide the following information:

- auth token (dynamic or static). Ex: `tctl nodes add --ttl=5m --roles=node | grep "invite token:" | grep -Eo "[0-9a-z]{32}"`
- CA pin
- address of the authentication server

## Role Variables

These are the default variables with their default values as defined in `defaults/main.yml`

```
teleport_version
```
The version of teleport to install. See [teleport downloads](https://goteleport.com/teleport/download/) for available versions.

```
teleport_architecture
```
Change `teleport_architecture` any of the following:

- `arm-bin` if you are running on ARMv7 (32-bit) based devices.
- `arm64-bin` if you are running on ARM64/ARMv8 (64-bit) based devices.
- `amd64-bin` if you are running on x86_64/AMD64 based devices.
- `386-bin` if you are running on i386/Intel based devices.

```
teleport_config_template
```
The template to use for the teleport configuration file. The default is `templates/default_teleport.yaml.j2`. It contains a basic configuration that will enable the SSH service and add a command label showing node uptime.

There are many [options available](https://goteleport.com/docs/setup/reference/config/) and you can substitute in your own template and add any variables you want.

```
teleport_service_template
```
The template to use for the teleport service file. The default is `templates/default_teleport.service.j2`. You can substitute in your own template and add any variables you want.

```
teleport_ca_pin
```
The CA pin to use for the teleport configuration. This is optional, but [recommended](https://goteleport.com/docs/setup/admin/adding-nodes/#untrusted-auth-servers).

```
teleport_config_path
```
The path to the teleport configuration file. The default is `/etc/teleport.yaml`.

```
teleport_auth_servers
```
The list of authentication servers to use for the teleport configuration. Examples are shown as defaults above.

```
backup_teleport_config
```
Runs a backup of the teleport configuration file before overwriting it. The default is `yes`. See [Upgrading Teleport](#upgrading-teleport) for more information.

```
teleport_control_systemd
```
Default `yes`. Controls if this role modifies the teleport service.

```
teleport_template_config
```
Default `yes`. Controls if this role modifies the teleport config file.

## Upgrading Teleport

When the role is run, it checks if the installed version matches the version specified in `teleport_version`. If different then it will download the latest version and install it.

When performing an upgrade, a backup of the current configuration file in `teleport_config_path` will be created and a new configuration file templated in its place. When doing this a `teleport_auth_token` and `teleport_ca_pin` do not need to be provided, as they are pulled from the existing configuration file, and then templated into the new configuration file.

This allows you to update values in the configuration file like labels and commands without having to store the auth token and ca pin.

This role reloads `teleport.service` after any of the following occur:

- Teleport is installed or updated
- Teleport configuration file is updated
- Teleport service file is updated

## Dependencies

None

## Example Playbook
For example to install teleport on a raspberry pi:
```
- hosts: all
  roles:
    - mdsketch.teleport
```

*Inside `templates/teleport.yaml.j2`*

```
teleport:
  auth_token: {{ teleport_auth_token }}
  ca_pin: {{ teleport_ca_pin }}
  auth_servers:
{% for auth_server in teleport_auth_servers %}
    - {{ auth_server }}
{% endfor %}
ssh_service:
  enabled: "yes"
  commands:
  - name: uptime
    command: [uptime, -p]
    period: 5m0s
proxy_service:
  enabled: "no"
auth_service:
  enabled: "no"
```

*Inside `templates/teleport.yaml.j2`*

```
teleport_auth_token: 1234
teleport_ca_pin: 1234
teleport_auth_servers:
    - "https://auth.example.com:443"
teleport_version: "7.3.3"
teleport_architecture: "arm-bin"
```

## License

MIT / BSD

## Author Information

This role was created in 2021 by Matthew Draws.
