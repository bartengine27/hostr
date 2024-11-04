### Install the Blazor Application

For this project, we will host the application (Blazor application) on [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-7.0). `Kestrel` is automatically included by publishing the application.

```shell
ansible-playbook webserver.yml -i hosts -K -vvv
```

:fire: Install the web server after installing the database: while installing the web server, the DB migration project will run and initialize the database.  