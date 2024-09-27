# Traefik reverse-proxy Ansible role

⚠️ **WARNING**: this role is now maintained in [mother-of-all-self-hosting/ansible-role-traefik](https://github.com/mother-of-all-self-hosting/ansible-role-traefik).

This is an [Ansible](https://www.ansible.com/) role which installs [Traefik](https://traefik.io/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on the [`com.devture.ansible.role.systemd_docker_base` role](https://github.com/devture/com.devture.ansible.role.systemd_docker_base).

## Usage

Example playbook:

```yaml
- hosts: servers
  roles:
    - role: galaxy/com.devture.ansible.role.systemd_docker_base

    - role: galaxy/com.devture.ansible.role.traefik

    - role: another_role
```

Example playbook configuration (`group_vars/servers` or other):

```yaml
devture_traefik_container_network: "{{ my_container_network }}"

devture_traefik_uid: "{{ my_uid }}"
devture_traefik_gid: "{{ my_gid }}"
```

## Security hardening

To avoid the Traefik container from mounting and using the Docker UNIX socket (`/var/run/docker.sock`) directly, you can also make it talk to the Docker API via TCP using [Tecnativa/docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy). The Traefik container can then run with reduced privileges (non-`root` user, [dropped capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities), etc).

To get this socket proxy installed, you can use the [`com.devture.ansible.role.container_socket_proxy` role](https://github.com/devture/com.devture.ansible.role.container_socket_proxy).

Here's some example configuration (e.g. `group_vars/servers`) which *optionally* wires them together:

```yaml
#
# Container Socket Proxy role configuration
#
devture_container_socket_proxy_enabled: true

devture_container_socket_proxy_uid: "{{ my_uid }}"
devture_container_socket_proxy_gid: "{{ my_gid }}"

# Traefik requires read access to the containers APIs to do its job
devture_container_socket_proxy_api_containers_enabled: true

#
# Traefik role configuration
#

# Base Traefik configuration here (see above).

devture_traefik_config_providers_docker_endpoint: "{{ devture_container_socket_proxy_endpoint if devture_container_socket_proxy_enabled else 'unix:///var/run/docker.sock' }}"

devture_traefik_container_additional_networks: |
  {{
    ([devture_container_socket_proxy_container_network] if devture_container_socket_proxy_enabled else [])
  }}

devture_traefik_systemd_required_services_list: |
  {{
    (['docker.service'])
    +
    ([devture_container_socket_proxy_identifier + '.service'] if devture_container_socket_proxy_enabled else [])
  }}
```
