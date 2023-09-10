# Introduction

Ansible scripts to setup *Proxmox* and/or *Azure* VMs/containers to install an example web site on prem/in the cloud.

* [Proxmox](#proxmox)
* [Azure](#azure)
* [Redis](#redis)
* [Database](#database)
* [References](#references)
* [TODO](#todo)

## Prerequisites

[see](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html)

Depending on your particular installation and/or test requirements, it may be relevant to configure an *DNS* server during the *Proxmox* installation: this will ensure that your *Proxmox* machine and VMs all are in the same subnet.

In this document/config it is assumed that you will use the following domain(s):

```text
# the ip address of the machine that (will) run(s) your web site, for example the ip address of proxmox vm 100
192.168.1.60 cars.be
```

Add the entries above in the hosts file on

* the machine running the Ansible installation scripts
* the clients (which run a browser) you use to visit the web site

Install the following *Ansible* roles:

```bash
ansible-galaxy install geerlingguy.redis
```

:fire: Microsoft has an ansible SQL Server role which supports RHEL, as we use Ubuntu for now, the Microsoft Ansible SQL Server role is not usable.

or write all dependencies to a `requirements.yml` file

```yaml
---
roles:
  - name: geerlingguy.redis
    type: galaxy
    source: https://galaxy.ansible.com

collections:
  # With just the collection name
  - my_namespace.my_collection    
```

and install with:

```bash
ansible-galaxy role install -r requirements.yml #what we use
ansible-galaxy collection install -r requirements.yml
```

## Secrets

Secrets are stored in env variables in files

* .env
* .env.prod
* .env.acc
* .env.dev

Keep your *PROD*, *ACC* and *DEV* secrets in the corresponding files and copy to `.env` before you start the ansible scripts. `source` .env Before executing your first Ansible script.

:fire: Obviously, none of the `env` files belongs in your repo.

Your `env` file should at least contain:

```bash
#!/bin/bash
export SSH_PUB_KEY="ssh-rsa AAAAB.... your_name@your_machine"
```

If you are unsure about your public key, check `~/.ssh/id_rsa_pub`.

## Proxmox

### Proxmox VM setup

As far as I know, *Ansible* is not the tool of choice to setup *VM*s on cloud/datacenter solutions like *Azure*, *Proxmox*, etc. Nevertheless, as an excercise, I've chosen to setup my *Azure* and *Proxmox* *VM*s with *Ansible*.  

*Ansible* Uses an `inventory` to define the machines (*ip addresses*) and machine roles to install/setup your software infrastructure. As we start with a clean install (no existing *VM*s), it is rather difficult to define an `inventory`.  

Therefore, we start with a play `proxmox.cars.be` which has a [vars file](./proxmox.cars.be/vars/main.yml) defining the *inventory*, for example:

```yml
proxmox_vms:
 - ip: 192.168.1.60
   id: 600
   group:
    - "webserver"
 - ip: 192.168.1.61
   id: 601
   group:
    - "redis"
 - ip: 192.168.1.62
   id: 602
   group:
    - "database"
```

On the other hand, it is probably wise to structure/group *ip addresses* and *vm id*s under `group` which is more aligned with an *Ansible inventory* (and easier to maintain). :fire: That's for another version.  

After running the [proxmox.cars.be](./proxmox.cars.be/) play, we can use the initialized *VM*s as defined in the [vars file](./proxmox.cars.be/vars/main.yml) as the basis for the *Ansible* `ìnventory`.

The [hosts](./hosts) file contains the `inventory` for the other *Ansible* plays. :fire: At this moment, the [hosts](./hosts) file is not generated from the [proxmox.cars.be](./proxmox.cars.be/) play, it is probably wise to change that in future versions.  

### Proxmox hosts file

You can pass a hosts file to the `ansible-playbook` command with:

```bash
-i ./hosts
```

### Uploading Proxmox VM(s) templates

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_upload -vvv
```

:fire: If this tag fails, you may have to upgrade your community.general modules, see [issue](https://github.com/ansible-collections/community.general/issues/6974):

```bash
ansible-galaxy collection install community.general
```

you should have at least version 7.2.1

```bash
ansible-galaxy collection list
```

and look for community.general.

:fire: You probably have two installs of community.general, if you'd like to run the system-wide install:

```bash
sudo ansible-galaxy collection install community.general
```

### Installing Proxmox VM(s)

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_init -vvv
```

### Stopping Proxmox VM(s)

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_stop -vvv
```

### Removing Proxmox VM(s)

Before removing a Proxmox VM, first stop the Proxmox VM, so execute the [stopping proxmox](#stopping-proxmox-vms) command or add tag vm_stop before the *vm_remove* tag in the following command:

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_remove -vvv
```

### SSH into the Proxmox VM

```bash
ssh root@192.168.1.60 -i /home/user/.ssh/authorized_keys
```

## Azure

TODO

## Redis

### Install Redis

```bash
ansible-playbook redis-setup.yml -i hosts -K -vvv
```

:fire: Note the `hosts` file, at the moment, only ip address are used in the host file. In an ideal case, hostnames as setup on *Proxmox* should be used grouped by labels redis, webserver, etc.

## Database

### Install SQL Server

```bash
ansible-playbook database-setup.yml -i hosts -K -vvv
```

## References

* (Playbooks directory)[https://charlesreid1.com/wiki/Ansible/Directory_Layout/Details]

## TODO

* ssh-keygen: add fingerprints automatically
* all passwords to env variables
* Azure
* align the [vars](./proxmox.cars.be/vars/main.yml) file with an *Ansible* `inventory`
* generate the *Ansible* `inventory` file or use an `inventory` file as a *vars.yml* file to setup *VM*s (if possible)
