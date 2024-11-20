### Install SQL Server

```shell
ansible-playbook mssql.yml -i hosts -K -vvv
```

After installing your DB server, you can SSH into the machine and open an SQL shell with (assuming you use password `p@55w0rD`):

```shell
sqlcmd -S 127.0.0.1 -U sa -P p@55w0rD -C
```

### Install Postgresql Server

Use the `hostr.postgresql` role in a playbook as follows:

```yml
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
