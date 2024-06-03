Cross-Site Request Forgery (CSRF) is a security threat where an attacker tricks a user into executing unwanted actions on a web application in which they're authenticated. If the victim is a regular user, a successful CSRF attack can force them to perform state-changing requests like transferring funds, changing their email address, and so forth. If the victim has an administrative account, CSRF can compromise the entire web application. CSRF exploits the trust that a site has in the user's browser, and unlike Cross-Site Scripting (XSS), which exploits the trust a user has in a particular site, CSRF exploits the trust that a site has in the user's browser.

In a distributed web application architecture, particularly one that scales horizontally as in this setup, requests from the same user can be routed to different servers across multiple requests due to load balancing. This poses a challenge for CSRF protection mechanisms that rely on keeping track of state, such as synchronizer tokens or double submit cookies, because the server handling a subsequent request might not have access to the tokens generated by another server on a previous request.

REDIS, as a fast, in-memory data store offers a solution for storing CSRF protection keys in such a distributed setup. 

In the drawing above, REDIS was added explicitely to stress the impact of the distributed nature and horizontal scalability of the setup, i.e. its non-trivial impact on CSRF. More details are available [on the cars.be repo](https://github.com/bartengine27/cars.be/tree/abp_8).

### Install REDIS

```shell
ansible-playbook redis.yml -i hosts -K -vvv
```

:fire: Note the `hosts` file, at the moment, only ip address are used in the host file. In an ideal case, hostnames as setup on *Proxmox* should be used grouped by labels redis, webserver, etc.

:fire: If you get an error like

```shell
 "msg": "Unable to start service redis-server: Job for redis-server.service failed because the control process exited with error code.\nSee \"systemctl status redis-server.service\" and \"journalctl -xe\" for details.\n"
```

after running the redis playbook, you probably forgot to `source .env`.  