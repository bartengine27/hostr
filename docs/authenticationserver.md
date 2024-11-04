### Install Authentication Server

`ABP` supports [IdentityServer4](https://github.com/IdentityServer/IdentityServer4) and [OpenIddict](https://github.com/openiddict/openiddict-core). As the `ABP` startup templates support *OpenIddict* since *ABP v6.0.0* we will only support `OpenIddict`.  

```shell
ansible-playbook authentication.yml -i hosts -K -vvv
```

After installing the authentication server, you can request a client credentials (server to server), check the documentation and example [in the cars.be repo](https://github.com/bartengine27/cars.be/tree/abp_8).
