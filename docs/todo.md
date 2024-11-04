* move proxmox playbook to a different "folder"
* generate the *Ansible* `inventory` file or use an `inventory` file as a *vars.yml* file to setup *VM*s (if possible)
* integrate [prometheus alerts](https://github.com/prometheus/alertmanager) or [grafana alerts](https://grafana.com/docs/grafana/latest/alerting/fundamentals/alertmanager/) or ...
* [enable tcpdump in a container](https://cloud.garr.it/support/kb/general/enableTcpdumpInLXCContainer/)
https://netsplit.uk/posts/2022/10/19/remote_ovh_lab/
* pfSense `pfctl -d` and `pfctl -e` to disable/enable the packet filter
* add DNS
* remove cert generation outside of ca/caclient roles
