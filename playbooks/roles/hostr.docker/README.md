I have extracted your Ansible Docker role. Based on the standard Ansible role structure and the contents of the extracted files, here is a suggested `README.md` for your role:

```markdown
# hostr.docker

## Overview

This Ansible role installs and configures Docker on a target host. It ensures that Docker is installed, configured with appropriate settings, and that the service is enabled and running.

## Requirements

This role requires the target host to be a supported Ubuntu distribution with access to the internet for package installations.

## Role Variables

The following variables are available for this role and can be customized to suit your needs. Default values are set in `defaults/main.yml`.

```yaml
# defaults/main.yml
docker_users: []
docker_daemon_options:
  {
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "100m"
    }
  }
docker_package_state: present
docker_service_state: started
docker_service_enabled: true
```

http_proxy: "{{ lookup('ansible.builtin.env', 'HTTP_PROXY') }}"
https_proxy: "{{ lookup('ansible.builtin.env', 'HTTPS_PROXY') }}"
no_proxy: "{{ lookup('ansible.builtin.env', 'NO_PROXY') }}"

docker_default_address_pools_base: 172.17.0.0/16
docker_default_address_pools_size: 20
docker_log_driver: json-file
docker_log_opts_max_size: 200m
docker_log_opts_max_file: 5

| Variable                | Default Value                                                              | Description                                           |
|-------------------------|----------------------------------------------------------------------------|-------------------------------------------------------|
| `http_proxy`            | `[]`                                                                       | HTTP proxy used by docker.                            |
| `https_proxy`           | `[]`                                                                       | HTTPS proxy used by docker.                           |
| `no_proxy`              | `[]`                                                                       | Do not use proxy.                                     |
| `docker_default_address_pools_base` | `[]`                                                           | The IP subnet to choose Docker container IP address from. |
| `docker_default_address_pools_size` | `[]`                                                           | The size of the IP subnet.                            |
| `docker_log_driver`     | `[]`                                                                       | The Docker log driver.                                |
| `docker_log_opts_max_size`          | `[]`                                                           | The maximum size of a Docker log file                 |
| `docker_log_opts_max_file`          | `[]`                                                           | The maximum number of log files                       |
| `docker_users`          | `[]`                                                                       | List of users to be added to the `docker` group.      |

## Dependencies

This role has no dependencies on other roles.

## Example Playbook

Below is an example of how to use this role in a playbook:

```yaml
- name: Install and configure Docker
  hosts: all
  become: true
  roles:
    - role: hostr.docker
      vars:
        docker_users:
          - user1
          - user2
        docker_log_driver: json-file
        docker_log_opts_max_size: 200m
        docker_log_opts_max_file: 5
```

## License

MIT

## Author Information

This role was created by [Bart](https://engine27.be).