### Install REST API

```shell
ansible-playbook abprestapi.yml -i hosts -K -vvv
```

The *REST API* contains a custom metric and controller, open the *REST API* interface (`Swagger`) and call `/api/car/increment` to increment the custom metric. If you'd like to monitor this metric in real-time, *SSH* into the [HTTP server](./../playbooks/hosts) and run:  

```shell
#if nessary, install dotnet-counters
#dotnet tool install --global dotnet-counters
dotnet-counters monitor --name Be.Cars.HttpApi.Host --counters Be.Cars.Metrics.CustomMetrics
```
