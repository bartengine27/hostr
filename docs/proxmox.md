### Proxmox VM setup

As far as I know, *Ansible* is not necessarily the tool of choice to setup *VM*s on cloud/datacenter solutions like *Proxmox*, etc. Nevertheless, as an excercise, I've chosen to setup my *Proxmox* *VM*s with *Ansible*.  

*Ansible* Uses an `inventory` to define the machines (*ip addresses*) and machine roles to install/setup your software infrastructure. As we start with a clean install (no existing *VM*s), it is rather difficult to define an `inventory`.  

Therefore, we start with a play [proxmox](./../playbooks/proxmox.yml) which uses [role](./../playbooks/roles/hostr.proxmox/README.md) and [vars file](./../playbooks/roles/hostr.proxmox/vars/main.yml) defining the *inventory*, for example:

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

On the other hand, it is probably wise to structure/group *ip addresses* and *vm id*s under `group:` which is more aligned with an *Ansible inventory* (and easier to maintain). :fire: That's for another version.  

After running the [proxmox](./../playbooks/proxmox.yml) play, we can use the initialized *VM*s as defined in the [vars file](./../playbooks/roles/hostr.proxmox/vars/main.yml) as the basis for the *Ansible* `ìnventory`.  

Running the [proxmox](./../playbooks/proxmox.yml) play is detailed in [Uploading templates](#uploading-proxmox-vms-templates) and [Installing VMs on Proxmox](#installing-proxmox-vms).  

:fire: The [hosts](./../playbooks/hosts) file contains the `inventory` for the other *Ansible* plays. :fire: At this moment, the [hosts](./../playbooks/hosts) file is not generated from the [proxmox](./../playbooks/proxmox.yml) play, it is probably a good idea to change that in future versions.  

### Uploading Proxmox VM(s) templates

```shell
ansible-playbook proxmox.yml -K --tags=vm_upload -vvv
```

:fire: If this tag fails, you may have to upgrade your community.general modules, see [issue](https://github.com/ansible-collections/community.general/issues/6974):

```shell
ansible-galaxy collection install community.general
```

you should have at least version 7.2.1

```shell
ansible-galaxy collection list
```

and look for community.general.

:fire: You probably have two installs of community.general, if you'd like to run the system-wide install:

```shell
sudo ansible-galaxy collection install community.general
```

### Installing Proxmox VM(s)

```shell
ansible-playbook proxmox.yml -K --tags=vm_init -vvv
```

### Addding Proxmox VM(s) to known_hosts

If you removed existing VMs/Containers or are building new VMs/Containers, you should remove the old entries in `known_hosts` and add new entries:

```shell
ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook knowhosts-setup.yml -i hosts -K -vvv
```

When you observe an additional line being added to your known_hosts file after connecting to a server via SSH, even after manually adding the server's fingerprint with ssh-keyscan or with `ansible-playbook knowhosts-setup.yml`, it typically relates to how SSH handles and verifies the identity of the servers it connects to. There are a few reasons why this might happen:

1. **Different Key Types**
If you initially add the server's `ED25519` key fingerprint to your known_hosts using ssh-keyscan -H -t ed25519 ip_address, but the server is also configured to use another type of SSH key (e.g., RSA, ECDSA), the SSH client might add the fingerprint of this additional key to the known_hosts file upon the first connection. This occurs because your initial scan and add operation only included the ED25519 key, and upon connection, SSH automatically adds any other keys presented by the server that weren't already in known_hosts.

2. **Hostname and IP Address Entries**  
Another common reason is the difference in how you reference the server in your ssh command versus what was initially scanned. For instance, if you scanned the IP address and then used the hostname (or vice versa) to connect, SSH treats these as separate entries. SSH distinguishes between IP addresses and hostnames because they can technically present different keys (consider virtual hosts or shared IP scenarios). As a result, SSH might add a new line for the same server under its different identifier (IP or hostname).

3. **SSH Configuration and Aliases**  
Your SSH client's configuration might influence how known_hosts is managed. For example, if you use an SSH config file (~/.ssh/config) with aliases or specific host entries that define hostname patterns or specific key types, your SSH client might treat connections that match different patterns as distinct, even if they ultimately resolve to the same server.

4. **Port Forwarding or Jump Hosts**  
If your connection involves port forwarding or using a jump host (also known as a bastion host), the SSH client may add entries for these intermediate steps. This is more likely in complex networking setups where direct connections to the target server are not possible without going through intermediary servers.

Troubleshooting Tips:

* **Review the known_hosts file**: Compare the entries before and after the connection. Look for differences in the key types, hostnames/IP addresses, or additional details that might explain the new entry.
* **Use verbose mode with SSH**: Connecting with `ssh -v root@ip_address` can provide detailed logs that explain what keys are being checked, offered, and added to known_hosts. This might give you a clearer picture of why the additional line is added.
* **Check SSH client configuration**: Review your SSH client's configuration file (if you have one) for any settings that might affect how known_hosts entries are managed or how connections are established.

Understanding the exact reason requires examining the specifics of your SSH setup, the server configuration, and the entries in known_hosts.

For our particular setup, we can edit the server config `sshd_config` file on the server we connect to to enforce the usage of `ED25519` keys:

```ini
#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
PubkeyAcceptedKeyTypes ssh-ed25519
```

in addition, make sure that folder `/run/sshd` exists on the server:

```shell
mkdir /run/sshd
chmod 0755 /run/sshd
systemctl restart sshd
```

Clients, however, should now support `ED25519`, i.e. use:

```shell
ssh-keygen -t ed25519 -C you@cars.be
```

If you now connect with:

```shell
ssh -v root@192.168.1.68
```

you will notice in the output that ssh will attempt different keys, for example:

```text
debug1: Will attempt key: /home/you/.ssh/id_rsa RSA SHA256:....
debug1: Will attempt key: /home/you/.ssh/id_ecdsa
debug1: Will attempt key: /home/you/.ssh/id_ecdsa_sk
debug1: Will attempt key: /home/you/.ssh/id_ed25519 ED25519 SHA256:....
debug1: Will attempt key: /home/you/.ssh/id_ed25519_sk
debug1: Will attempt key: /home/you/.ssh/id_xmss
debug1: Will attempt key: /home/you/.ssh/id_dsa
```

To conclude, the server decides about the security of the keys used by the client, therefore, we will configure all sshd daemons for `ED25519`:  

```shell
ansible-playbook sshdserver_setup.yml -i hosts --user root -K -vvv
```

:fire: User root is specified with `--user root`: the hosts we connect to have the `ED25519` key under `/root/.ssh/authorized_keys`, so we have to connect as `root` to use this key.  

### Stopping Proxmox VM(s)

```shell
ansible-playbook proxmox.yml -K --tags=vm_stop -vvv
```

### Starting Proxmox VM(s)

```shell
ansible-playbook proxmox.yml -K --tags=vm_start -vvv
```

if necessary, you can restart with

```shell
ansible-playbook proxmox.yml -K --tags=vm_restart -vvv
```

### Removing Proxmox VM(s)

Before removing a Proxmox VM, first stop the Proxmox VM, so execute the [stopping proxmox](#stopping-proxmox-vms) command or add tag vm_stop before the *vm_remove* tag in the following command:

```shell
ansible-playbook proxmox.yml -K --tags=vm_stop,vm_remove -vvv
```
