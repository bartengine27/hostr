### Install SQL Server

```shell
ansible-playbook mssql.yml -i hosts -K -vvv
```

After installing your DB server, you can SSH into the machine and open an SQL shell with (assuming you use password `p@55w0rD`):

```shell
sqlcmd -S 127.0.0.1 -U sa -P p@55w0rD -C
```