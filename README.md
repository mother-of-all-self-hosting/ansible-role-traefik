<!--
SPDX-FileCopyrightText: 2022 - 2024 Slavi Pantaleev

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Traefik reverse-proxy Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Traefik](https://traefik.io/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on the [`com.devture.ansible.role.systemd_docker_base` role](https://github.com/devture/com.devture.ansible.role.systemd_docker_base).

## Usage

Example playbook:

```yaml
- hosts: servers
  roles:
    - role: galaxy/com.devture.ansible.role.systemd_docker_base

    - role: galaxy/traefik

    - role: another_role
```

Example playbook configuration (`group_vars/servers` or other):

```yaml
traefik_container_network: "{{ my_container_network }}"

traefik_uid: "{{ my_uid }}"
traefik_gid: "{{ my_gid }}"
```

## Security hardening

To avoid the Traefik container from mounting and using the Docker UNIX socket (`/var/run/docker.sock`) directly, you can also make it talk to the Docker API via TCP using [Tecnativa/docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy). The Traefik container can then run with reduced privileges (non-`root` user, [dropped capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities), etc).

To get this socket proxy installed, you can use the [`ansible-role-container-socket-proxy` role](https://github.com/mother-of-all-self-hosting/ansible-role-container-socket-proxy).

Here's some example configuration (e.g. `group_vars/servers`) which *optionally* wires them together:

```yaml
#
# Container Socket Proxy role configuration
#
container_socket_proxy_enabled: true

container_socket_proxy_uid: "{{ my_uid }}"
container_socket_proxy_gid: "{{ my_gid }}"

# Traefik requires read access to the containers APIs to do its job
container_socket_proxy_api_containers_enabled: true

#
# Traefik role configuration
#

# Base Traefik configuration here (see above).

traefik_config_providers_docker_endpoint: "{{ container_socket_proxy_endpoint if container_socket_proxy_enabled else 'unix:///var/run/docker.sock' }}"

traefik_container_additional_networks_auto: |
  {{
    ([container_socket_proxy_container_network] if container_socket_proxy_enabled else [])
  }}

traefik_systemd_required_services_list_auto: |
  {{
    ([container_socket_proxy_identifier + '.service'] if container_socket_proxy_enabled else [])
  }}
```

## Development

You can optionally install [pre-commit](https://pre-commit.com/) so that simple mistakes are checked and noticed before changes are pushed to a remote branch. See [`.pre-commit-config.yaml`](./.pre-commit-config.yaml) for which hooks are to be executed.

See [this section](https://pre-commit.com/#usage) on the official documentation for usage.
