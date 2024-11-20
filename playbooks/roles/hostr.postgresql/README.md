Role Name
=========

This Ansible role installs and configures PostgreSQL on a target host. It ensures that PostgreSQL is installed, configured with default settings, and that the service is enabled and running after executing migration scripts.

Requirements
------------

This role requires the target host to be a supported Ubuntu or Rocky distribution with access to the internet for package installations. :fire: Currently, no version checks are executed to get a pre-defined PostgreSQL version.

Role Variables
--------------

The following variables are available for this role and can be customized to suit your needs. Default values are set in `defaults/main.yml`:

| Variable            | Default Value | Example Value         | Description                   |
|---------------------|---------------|------------------------------|------------------------|
| `db_name`           | ``            |  identity             | The name of yourdata tabase.  |
| `db_user`           | ``            |  identity_user        | The user of your database     |
| `db_password`       | ``            |  Testtest,0           | The password of your database.|
| `db_migration`      | ``            |  identity_template.sl | The migration script for your database.|

Dependencies
------------

This role has no dependencies on other roles.

Example Playbook
----------------

Below is an example of how to use this role in a playbook:

```yaml
---
- name: "Setup of the postgresql backend"
  hosts: db
  remote_user: root
  roles:
    - role: hostr.postgresql
      vars:
        databases:
          - db_name: identity
            db_user: identity_user
            db_password: Testtest,0
            db_migration: identity_migration.sql
          - db_name: keycloak
            db_user: keycloak_user
            db_password: Testtest,0
            db_migration: keycloak_migration.sql
```

License
-------

MIT

Author Information
------------------

This role was created by [Bart](mailto:bart@engine27.be).