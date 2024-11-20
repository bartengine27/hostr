Role Name
=========

This Ansible role installs and configures Microsoft SQL Server on a target host. It ensures that SQL Server is installed, configured with appropriate settings, and that the service is enabled and running.

Requirements
------------

This role requires the target host to be a supported Ubuntu distribution with access to the internet for package installations. :fire: Currently, no Ubuntu version checks are executed to get the correct SQL Server installer.

Role Variables
--------------

The following variables are available for this role and can be customized to suit your needs. Default values are set in `defaults/main.yml`:

| Variable                | Default Value     | Example Value                                         | Description
                      |
|-------------------------|-------------------|---------------------------------------------------------|-------------------------------------------------------|
| `database_password`           | ``                 |  password_123                      | SQL Server sa password.

Dependencies
------------

This role has no dependencies on other roles.

Example Playbook
----------------

Below is an example of how to use this role in a playbook:

```yaml
---
- name: SQLServer setup playbook
  hosts: database
  remote_user: root
  roles:
     - hostr.mssql
```

License
-------

MIT

Author Information
------------------

This role was created by [Bart](mailto:bart@engine27.be).