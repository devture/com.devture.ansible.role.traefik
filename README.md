# Traefik reverse-proxy Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Traefik](https://traefik.io/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base).

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
