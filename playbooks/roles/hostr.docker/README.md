# hostr.docker

## Overview

This Ansible role installs and configures Docker on a target host. It ensures that Docker is installed, configured with appropriate settings, and that the service is enabled and running.

## Requirements

This role requires the target host to be a supported Ubuntu distribution with access to the internet for package installations.

## Role Variables

The following variables are available for this role and can be customized to suit your needs. Default values are set in `defaults/main.yml`: 

| Variable                | Default Value     | Example Value                                         | Description                                           |
|-------------------------|-------------------|---------------------------------------------------------|-------------------------------------------------------|
| `http_proxy`            | ``                 |  http://my_proxy:my_proxy_port/                      | HTTP proxy used by docker.                            |
| `https_proxy`           | ``                 |  http://my_proxy:my_proxy_port/                      | HTTPS proxy used by docker.                           |
| `no_proxy`              | ``                 |  localhost,127.0.0.0/8,you.local,registry.you.local  | Do not use proxy.                                     |
| `docker_default_address_pools_base` | ``     |  172.17.0.0/16                                       | The IP subnet to choose Docker container IP address from. |
| `docker_default_address_pools_size` | ``     |  20                                                  | The size of the IP subnet.                            |
| `docker_log_driver`     | ``                 |  json-file                                           | The Docker log driver.                                |
| `docker_log_opts_max_size`          | ``     |  200m                                                | The maximum size of a Docker log file                 |
| `docker_log_opts_max_file`          | ``     |  5                                                   | The maximum number of log files                       |
| `docker_users`          | ``                 |    - name: user1 <br/> pubkey: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIID42JAWkOxmP1WjuVVPNLeqsEq3KiiSHlWALLd22u68 bart@engine27.be <br/> - name: user2 <br/> pubkey: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIID42JAWkOxmP1WjuVVPNLeqsEq3KiiSHlWALLd22u68 bart@engine27.be                                      | List of users to be added to the `docker` group.      |

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

## TODO

`docker_users`

## Known Issues

* `sudo user1`: sudo: setrlimit(RLIMIT_CORE): Operation not permitted