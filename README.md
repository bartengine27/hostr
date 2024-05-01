# Introduction

* [Introduction](#introduction)
  * [Prerequisites](#prerequisites)
    * [Proxmox prerequisites](#proxmox-prerequisites)
      * [Hyper-V install](#hyper-v-install)
      * [Bare metal install](#bare-metal-install)
      * [Promox API user](#promox-api-user)
    * [Ansible prerequisites](#ansible-prerequisites)
    * [Secrets prerequisites](#secrets-prerequisites)
    * [Ansible role settings](#ansible-role-settings)
    * [Git/SSH prerequisites](#gitssh-prerequisites)
    * [Proxmox Firewall Preparation](#proxmox-firewall-preparation)
    * [Bridges](#bridges)
    * [SSH into the Proxmox VM](#ssh-into-the-proxmox-vm)
  * [Proxmox](#proxmox)
    * [Proxmox VM setup](#proxmox-vm-setup)
    * [Proxmox hosts file](#proxmox-hosts-file)
    * [Uploading Proxmox VM(s) templates](#uploading-proxmox-vms-templates)
    * [Installing Proxmox VM(s)](#installing-proxmox-vms)
    * [Addding Proxmox VM(s) to known\_hosts](#addding-proxmox-vms-to-known_hosts)
    * [Stopping Proxmox VM(s)](#stopping-proxmox-vms)
    * [Starting Proxmox VM(s)](#starting-proxmox-vms)
    * [Removing Proxmox VM(s)](#removing-proxmox-vms)
  * [REDIS](#redis)
    * [Install REDIS](#install-redis)
  * [Authentication Server](#authentication-server)
    * [Install Authentication Server](#install-authentication-server)
  * [REST API](#rest-api)
    * [Install REST API](#install-rest-api)
  * [Proxy](#proxy)
    * [Install NGINX Proxy](#install-nginx-proxy)
  * [Database](#database)
    * [Install SQL Server](#install-sql-server)
  * [Web server](#web-server)
    * [Install the Blazor Application](#install-the-blazor-application)
  * [Application Performance Monitoring](#application-performance-monitoring)
    * [Prometheus server](#prometheus-server)
      * [Install Prometheus server](#install-prometheus-server)
    * [Collector](#collector)
      * [Install InfluxDB](#install-influxdb)
    * [Grafana server](#grafana-server)
      * [Install Grafana server](#install-grafana-server)
  * [Certificate Authority](#certificate-authority)
    * [Install Certificate Authority](#install-certificate-authority)
    * [Generate Client Certificates](#generate-client-certificates)
  * [HTTPS Offloading](#https-offloading)
    * [NGINX](#nginx)
      * [Log format](#log-format)
      * [Load balancing](#load-balancing)
      * [HTTPS configuration](#https-configuration)
      * [Access logging](#access-logging)
      * [Location block and proxy settings](#location-block-and-proxy-settings)
      * [Optional websocket support](#optional-websocket-support)
  * [References](#references)
  * [TODO](#todo)

While developing it may be useful to have a setup which mirrors your final production setup as close as possible. In practice, hardware and/or time are not always available on time to setup a test or acception environment which mirrors production. Therefore, having a setup that you can easily (re-)deploy on a (local) virtual environment may be your way out.  

This project contains *Ansible* scripts to setup *Proxmox* VMs/containers to install a web site on prem or in the cloud to ease/speed-up development. The *Ansible* scripts in this project assume that you need servers to host the following roles:

* Web Server
* Cache
* Database
* Identity Server  
* REST API endpoints
* Prometheus monitoring
* Grafana dashboards

## Prerequisites

### Proxmox prerequisites

If you'd like to deploy on Proxmox, first install Proxmox on a host. Your host is a bare metal machine or a virtual machine with nesting enabled. [Qemu](https://www.qemu.org/) For instance, allows to nest VMs. **TBC**  

Depending on your particular installation and/or test requirements, it may be relevant to configure an *DNS* server during the *Proxmox* installation: this will ensure that the machine running *Proxmox* (bare metal or VM) and the VMs running on *Proxmox* all are in the same subnet.

In this document/config it is assumed that you will use the following domain(s):

```text
# the ip address of the machine that (will) run(s) your web site, for example the ip address of proxmox vm 100
192.168.1.60 cars.be
```

Add the entries above in the hosts file on

* the machine running the Ansible installation scripts
* the clients you use to visit the web site

If all has gone well, you should have (after running the *Ansible* scripts) a setup like:

![setup](./Proxmox%20setup.drawio.png)

#### Hyper-V install

You can install Proxmox on Windows Hyper-V. Create the virtual machine with the standard procedure and execute the following command in Powershell

```Powershell
Set-VMProcessor -VMName <Name of your Proxmox VM> -ExposeVirtualizationExtensions $true
``` 

to switch on Hyper-V's nested virtualization. Download and mount the Proxmox ISO, after booting, click in the VM screen and tap on Enter to start the install. If your install hangs at `Trying to detect country`, disable your internet connection and restart the installer.  

After installing Proxmox on Hyper-V, you will notice that calling an endpoint on your Hyper-V VM from WSL2 doesn't work. Open the Hyper-V Manager and next the Virtual Switch Manager

![Switches](./wsl_hypervvm_switches.png)

and not the two different virtual switches. As documented [on](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-subsystem-for-linux-2-addressing-traffic-routing-issues/ba-p/1764074) we need to enable forwarding across both virtual switches:

```Powershell
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (Default Switch)' -or $_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))'} | Set-NetIPInterface -Forwarding Enabled
```

:fire: Note the **vEthernet(...)** in the IP interface aliases, if you are not sure about the aliases, first run

```Powershell
Get-NetIPInterface
```

and check the second column.  

#### Bare metal install

Follow the default installation procedure [on](https://www.proxmox.com/en/proxmox-virtual-environment/get-started).

#### Promox API user

Before using the Ansible scripts, you will need a Proxmox API user:

* Log into the Proxmox Web UI
* Select datacenter from the left menu
* Select users fromf the left sub-menu
* We will use the default root@pam user
* Select API tokens from the left sub-menu
* Click add
  * User: root@pam
  * Token ID: input a random string, for [example](https://www.random.org/passwords/?num=1&len=24&format=plain&rnd=new) and record the token ID
  * record the token secret

After adding the user and API token, select Datacenter and click on Permissions and Add API Token Permission. Choose

* Path: /
* API Token: root@pam<tokenID>
* Role: Administrator
* Propagate: select

### Ansible prerequisites

As we are using [Ansible](https://www.ansible.com/) to automate our install and deploy, you will have to meet the following minimum requirements on the hosts which will run the *Ansible* scripts: [Ansible minimum requirements](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html).

For this project, we will need some extra dependencies. Install the following *Ansible* role(s):

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
ansible-galaxy role install -r requirements.yml
ansible-galaxy collection install -r requirements.yml
```

Add additional Python dependencies with:

```bash
pip install proxmoxer
```

### Secrets prerequisites

Secrets are stored in env variables in files

* .env
* .env.prod
* .env.acc
* .env.dev

Keep your *PROD*, *ACC* and *DEV* secrets in the corresponding files and copy to `.env` before you start the ansible scripts. `source` .env Before executing your first Ansible script.

:fire: Obviously, none of the `env` files belongs in your repo.

Your `env` file should at least contain your public key to connect with ssh to the VMs:

```bash
#!/bin/bash
export SSH_PUB_KEY="ssh-rsa AAAAB.... your_name@your_machine"
```

If you are unsure about your public key, check `~/.ssh/id_rsa_pub`.

### Ansible role settings

Each role has its variables (settings) defind in `role_folder/defaults/main.yml`. Variables are read from env variables:

```yaml
project_name: "{{ lookup('ansible.builtin.env', 'PROJECT_NAME') }}"
```

You can - as usual in Ansible - easily override the default variables.  

### Git/SSH prerequisites

When managing source code across virtual machines (VMs) on platforms like Proxmox, accessing source code from version control systems may have a security impact. A common practice involves cloning repositories using Git over SSH, authenticated via private SSH keys. However, storing private keys directly on VMs, especially those hosted on third-party servers, raises potential significant security risks. Instead, SSH agent forwarding emerges as a safer alternative.  

SSH agent forwarding allows users to access a remote machine through SSH without placing private SSH keys on the server itself. Instead, the SSH authentication request is forwarded back to the user's local machine where the SSH agent resides. This means the private key is never exposed to the network or stored on the remote server, obviously enhancing security.

You can setup SSH agent forwarding on your local machine with:

* **Enable SSH Agent on Local Machine**: Before initiating a connection, ensure the SSH agent is running on your local machine and your private key is added. This can be done using:

  ```shell
  eval $(ssh-agent -s)  
  # note the full path, if you are using WSL and the windows agent (1password, etc.) you have to be explicit about "which ssh config" you are changing (Ansible does not use the 1password Windows agent)
  /usr/bin/ssh-add # add all keys under ~/.ssh/
  /usr/bin/ssh-add -l # list all keys
  ```  

* **Configure SSH to Use Agent Forwarding**: Modify the SSH configuration file (~/.ssh/config) on your local machine to enable agent forwarding. You can specify this globally or for specific hosts:

  ```shell
  Host 192.168.1.*    
    ForwardAgent yes
  ```

* **Connect to the VM**: Once configured, you can connect to the VM using SSH. The agent forwarding will carry your authentication from the local machine to the VM:

  ```shell  
  /usr/bin/ssh-add # note the full path, if you are using WSL and the windows agent (1password, etc.) you have to be explicit about "which ssh config" you are changing (Ansible does not use the 1password Windows agent)
  /usr/bin/ssh -v -T git@github.com
  #Hi your_account! You've successfully authenticated, but GitHub does not provide shell access.
  /usr/bin/ssh root@192.168.1.60 # SSH into your host, you can test this once your hosts are "online and reachable"
  echo $SSH_AUTH_SOCK #response should not be empty
  #/tmp/ssh-n9uZGGjGAC/agent.2806
  ssh -v -T git@github.com # on your host, you probably are not using WSL and the windows agent  
  #Hi username You've successfully authenticated, but GitHub does not provide shell access.
  ```

* **Clone Repository Using SSH on the VM**: On the VM, use Git to clone the repository as usual. The SSH agent will authenticate using your local machine’s private key, as an example (will be executed in Ansible scripts):  

  ```shell
  git clone git@github.com:user/repository.git
  ```

Some advantages of SSH agent forwarding:

* **Enhanced Security**: The primary advantage is security. Your private SSH keys remain secure on your local machine, reducing the risk of exposure through VM compromise.
* **Ease of Use**: Once set up, the process is transparent to the user. You can seamlessly authenticate to Git repositories without managing multiple keys on different VMs.

Some drawbacks and potential considerations:

* **Security Risks with Misconfiguration**: If not configured properly, SSH agent forwarding can expose your local SSH agent to remote machines, which could be a vector for an attack if the remote machine is compromised.
* **Dependency on Local Machine**: Your local machine must be online and accessible whenever operations requiring SSH authentication are performed, potentially limiting flexibility.
* **Complex Setup**: The initial setup requires understanding of SSH configurations and might be more complex compared to simply copying keys to servers.

Some best practices:

* **Limit Forwarding**: Only enable agent forwarding to trusted hosts to mitigate the risk of exposing your SSH agent to a compromised server.
* **Monitor Sessions**: Keep an eye on active SSH sessions that utilize agent forwarding to ensure they are legitimate and necessary.
* **Security Hardening**: Regularly update and patch both local and remote systems to protect against vulnerabilities.

:fire: If your source code is hosted on a public repository, you can off course skip the steps above.  

### Proxmox Firewall Preparation

Before you can install a firewall, you have to make sure that at least one of your VMs an access the Internet, all
other VMs on Proxmox have to be able to communicate with each other in a LAN only. No Internet connectivity is required for
the "non firewall" VMs.  

In order to arrange this setup, we will need two bridges:

* one bridge which will *bridge* the incoming Internet traffic to the firewall and vice-versa
* one bridge which will allow the VMs to communicate within a LAN
* the VMs within the LAN use the LAN IP address of the firewall as gateway to return traffic over the firewall/VPN

A step-by-step guide to achieve this:

* You should have one existing bridge with a IPv4 which equals your public IP address (not the additional IP address)
* edit the MAC address of the network interface of the VM which has to access the Internet, enter the MAC address of the additional IP address
* Create a New Bridge for the Internal Network
  * Log into Proxmox VE Web Interface: Open your browser and access the Proxmox VE web interface by navigating to your Proxmox server's IP address.  
  * Navigate to 'System' -> 'Network': Here you will see your current network configuration, including any bridges and physical interfaces.
  * Create a New Bridge:
    * Click on "Create" and select "Linux Bridge".
    * Give the bridge a meaningful name, like vmbr1 (assuming vmbr0 is your existing bridge for the internet connection).
    * Assign it an IP address that fits within your internal network scheme (e.g., 192.168.1.1/16). If the Proxmox host doesn't need to communicate on this network, you may leave the IP address field empty.
    * Leave the Gateway field empty (the existing bridge has a Gateway)
    * Ensure the "Autostart" option is checked.
    * Leave the "Bridge ports" field empty if this bridge is for internal VM communication only.
    * Click "Create".
* Attach a Second Network Interface to Your VM
  * Select Your VM: In the Proxmox web interface, go to the "Server View", find the VM you want to configure, and click on it.
  * Add Network Device:
    * With the VM selected, go to the "Hardware" tab.
    * Click "Add" and choose "Network Device".
    * For "Model", you can choose "VirtIO (paravirtualized)" for better performance or "e1000" for broader compatibility.
    * Under "Bridge", select the new bridge you created (vmbr1).
    * Click "Add".
* Configure the Operating System Inside the VM: After attaching the second network interface, you need to configure the operating system inside the VM to use this new interface.  
  * Access Your VM: Log into your VM via the Proxmox console or SSH.
  * Identify the New Interface: Run ip a or ifconfig to list your network interfaces. You should see a new one (likely named eth1 or similar).
  * Configure the Network Interface: You'll need to edit the network configuration file or use a network manager to configure the interface. The exact steps vary depending on your Linux distribution, for Ubuntu:

  ```network
  # /etc/network/interfaces - create the file if it does not exist
  auto eth1
  iface eth1 inet static
      address 192.168.1.x
      netmask 255.255.0.0
      gateway 192.168.1.1
  ```

  For pfSense, you don't need to edit /etc/network/interfaces or /etc/netplan/..., it is sufficient to add the second network interface and assign its IP address in the pfSense UI.
* Restart the Network Service or the VM to apply changes.  

### Bridges

The terms *bridge* and *switch* are often used interchangeably in the context of networking, but they refer to devices that operate at the data link layer (Layer 2) of the OSI model.  

A bridge in Proxmox (or any other virtualization platform) is conceptually closer to a physical switch than to a traditional physical bridge in terms of its functionality and use case. Here's why:

* Virtual Bridges and Physical Switches: Similarities

  * **Multiple Connections**: Both virtual bridges in Proxmox and physical switches are designed to connect multiple devices (or virtual machines/containers in the case of Proxmox) within a network. They allow for the creation of network segments that can communicate internally and externally.
  * **Traffic Management**: Like a physical switch, a virtual bridge can manage and forward traffic between its connected interfaces based on MAC addresses, efficiently directing packets to their intended destinations within the virtual network or to external networks. 
  * **Advanced Networking Features**: While not as feature-rich as some high-end physical switches, virtual bridges in Proxmox can still offer several advanced networking features, such as VLAN tagging, which is a hallmark of switch capabilities.

* Differences from Physical Bridges

  * **Functionality**: Traditional physical bridges were primarily used to connect two network segments, with a focus on reducing collision domains and managing traffic between these segments. While a Proxmox bridge can perform a similar role in a virtual environment, it more closely mirrors the multiport, multipurpose functionality of switches by allowing numerous virtual machines and containers to connect to various networks.
  * **Port Density**: Physical bridges typically have a very limited number of ports (often just two), akin to the simplest form of segmentation. In contrast, a virtual bridge in Proxmox can handle connections from numerous VMs and containers simultaneously, much like a physical switch with many ports.

* Use in Virtualization

  * **Network Virtualization**: Virtual bridges play a crucial role in network virtualization, providing a platform for VMs and containers to communicate as if they were connected to a physical switch. This is essential for creating complex virtual network topologies that resemble physical network infrastructures.
  * **Flexibility and Scalability**: In virtual environments, bridges offer a level of flexibility and scalability that is more characteristic of switches. Administrators can dynamically adjust network configurations, add or remove VMs from networks, and implement security policies or VLANs without needing physical hardware changes.

In summary, while the terminology may suggest a direct analogy to physical bridges, in practice, the role of a bridge in Proxmox and other virtualized environments aligns more closely with the functionalities of a physical switch, especially concerning its ability to connect multiple devices and manage network traffic efficiently within a virtualized networking context.

For our setup, that becomes:

```mermaid
graph TB
  subgraph Internet
  end
  subgraph Proxmox
    direction TB

    subgraph vmbr0 [vmbr0<br>Public IPv4: 37.187.28.32<br>Gateway: 37.187.28.254]

    end

    subgraph VM500 [VM 500 - PfSense]
      direction TB
      eth0(eth0<br>MAC Additional IP)
      eth1(eth1<br>IP: 192.168.1.51)
      eth0 <--firewall--> eth1
    end

    subgraph vmbr1 [vmbr1<br>LAN Subnet: 192.168.1.1/16<br>Gateway: 192.168.1.51]

    end       

    subgraph LAN [LAN Subnet: 192.168.1.1/16<br>Gateway: 192.168.1.51]

    end

    vmbr0 --> eth0
    eth1 --> vmbr1
    vmbr1 --> LAN    
  end 
  Internet <---> vmbr0  
```

### SSH into the Proxmox VM

```bash
ssh root@192.168.1.60 -i /home/user/.ssh/authorized_keys
```

## Proxmox

### Proxmox VM setup

As far as I know, *Ansible* is not the tool of choice to setup *VM*s on cloud/datacenter solutions like *Proxmox*, etc. Nevertheless, as an excercise, I've chosen to setup my *Proxmox* *VM*s with *Ansible*.  

*Ansible* Uses an `inventory` to define the machines (*ip addresses*) and machine roles to install/setup your software infrastructure. As we start with a clean install (no existing *VM*s), it is rather difficult to define an `inventory`.  

Therefore, we start with a play `proxmox.cars.be` which has a [vars file](./playbooks/proxmox.cars.be/vars/main.yml) defining the *inventory*, for example:

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

After running the [proxmox.cars.be](./playbooks/proxmox.cars.be/) play, we can use the initialized *VM*s as defined in the [vars file](./playbooks/proxmox.cars.be/vars/main.yml) as the basis for the *Ansible* `ìnventory`.  

Running the [proxmox.cars.be](./playbooks/proxmox.cars.be/) play is detailed in [Uploading templates](###uploading-proxmox-vm(s)-templates) and [Installing VMs on Proxmox](###installing-proxmox-vm(s)).  

The [hosts](./playbooks/hosts) file contains the `inventory` for the other *Ansible* plays. :fire: At this moment, the [hosts](./playbooks/hosts) file is not generated from the [proxmox.cars.be](./playbooks/proxmox.cars.be/) play, it is probably a good idea to change that in future versions.  

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

### Addding Proxmox VM(s) to known_hosts

If you removed existing VMs/Containers or are building new VMs/Containers, you should remove the old entries in `known_hosts` and add new entries:

```bash
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

```bash
mkdir /run/sshd
chmod 0755 /run/sshd
systemctl restart sshd
```

Clients, however, should now support `ED25519`, i.e. use:

```bash
ssh-keygen -t ed25519 -C you@cars.be
```

If you now connect with:

```bash
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

```bash
ansible-playbook sshdserver_setup.yml -i hosts --user root -K -vvv
```

:fire: User root is specified with `--user root`: the hosts we connect to have the `ED25519` key under `/root/.ssh/authorized_keys`, so we have to connect as `root` to use this key.  

### Stopping Proxmox VM(s)

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_stop -vvv
```

### Starting Proxmox VM(s)

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_start -vvv
```

if necessary, you can restart with

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_restart -vvv
```

### Removing Proxmox VM(s)

Before removing a Proxmox VM, first stop the Proxmox VM, so execute the [stopping proxmox](#stopping-proxmox-vms) command or add tag vm_stop before the *vm_remove* tag in the following command:

```bash
ansible-playbook proxmox_ubuntu_vm-setup.yml -K --tags=vm_stop,vm_remove -vvv
```

## REDIS

Cross-Site Request Forgery (CSRF) is a security threat where an attacker tricks a user into executing unwanted actions on a web application in which they're authenticated. If the victim is a regular user, a successful CSRF attack can force them to perform state-changing requests like transferring funds, changing their email address, and so forth. If the victim has an administrative account, CSRF can compromise the entire web application. CSRF exploits the trust that a site has in the user's browser, and unlike Cross-Site Scripting (XSS), which exploits the trust a user has in a particular site, CSRF exploits the trust that a site has in the user's browser.

In a distributed web application architecture, particularly one that scales horizontally as in this setup, requests from the same user can be routed to different servers across multiple requests due to load balancing. This poses a challenge for CSRF protection mechanisms that rely on keeping track of state, such as synchronizer tokens or double submit cookies, because the server handling a subsequent request might not have access to the tokens generated by another server on a previous request.

REDIS, as a fast, in-memory data store offers a solution for storing CSRF protection keys in such a distributed setup. 

In the drawing above, REDIS was added explicitely to stress the impact of the distributed nature and horizontal scalability of the setup, i.e. its non-trivial impact on CSRF. More details are available [on the cars.be repo](https://github.com/bartengine27/cars.be/tree/abp_8).

### Install REDIS

```bash
ansible-playbook redis.yml -i hosts -K -vvv
```

:fire: Note the `hosts` file, at the moment, only ip address are used in the host file. In an ideal case, hostnames as setup on *Proxmox* should be used grouped by labels redis, webserver, etc.

:fire: If you get an error like

```bash
 "msg": "Unable to start service redis-server: Job for redis-server.service failed because the control process exited with error code.\nSee \"systemctl status redis-server.service\" and \"journalctl -xe\" for details.\n"
```

after running the redis playbook, you probably forgot to `source .env`.  

## Authentication Server

### Install Authentication Server

`ABP` supports [IdentityServer4](https://github.com/IdentityServer/IdentityServer4) and [OpenIddict](https://github.com/openiddict/openiddict-core). As the `ABP` startup templates support *OpenIddict* since *ABP v6.0.0* we will only support `OpenIddict`.  

```bash
ansible-playbook authentication.yml -i hosts -K -vvv
```

After installing the authentication server, you can request a client credentials (server to server), check the documentation and example [in the cars.be repo](https://github.com/bartengine27/cars.be/tree/abp_8).

## REST API

### Install REST API

```bash
ansible-playbook abprestapi.yml -i hosts -K -vvv
```

The *REST API* contains a custom metric and controller, open the *REST API* interface (`Swagger`) and call `/api/car/increment` to increment the custom metric. If you'd like to monitor this metric in real-time, *SSH* into the [HTTP server](./playbooks/hosts) and run:  

```bash
#if nessary, install dotnet-counters
#dotnet tool install --global dotnet-counters
dotnet-counters monitor --name Be.Cars.HttpApi.Host --counters Be.Cars.Metrics.CustomMetrics
```

## Proxy

### Install NGINX Proxy

```bash
ansible-playbook nginx.yml -i hosts -K -vvv
```

## Database

### Install SQL Server

```bash
ansible-playbook mssql.yml -i hosts -K -vvv
```

After installing your DB server, you can SSH into the machine and open an SQL shell with (assuming you use password `p@55w0rD`):

```bash
sqlcmd -S 127.0.0.1 -U sa -P p@55w0rD -C
```

## Web server

### Install the Blazor Application

For this project, we will host the application (Blazor application) on [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-7.0). `Kestrel` is automatically included by publishing the application.

```bash
ansible-playbook webserver.yml -i hosts -K -vvv
```

:fire: Install the web server after installing the database: while installing the web server, the DB migration project will run and initialize the database.  

## Application Performance Monitoring

In this project we will use [Prometheus](https://prometheus.io/) as it supports [OTLP (Open Telemetry Protocol)](https://opentelemetry.io/) which is used in the example code (web site and REST API). You can also export OTel (Open Telemetry) data to [Jaeger](https://www.jaegertracing.io/) or [Zipkin](https://zipkin.io/). For an overview, please check the following list of  [APM (Application Performance Monitoring) vendors](https://opentelemetry.io/ecosystem/vendors/).

```mermaid
flowchart TD;    
    WebServer[Web Server]-- ":5000/metrics" -->Prometheus;
    RESTAPI[REST API]-- ":5000/metrics" -->Prometheus;
    Prometheus-- ":9090" -->Grafana
```

:fire: The example code is written in dotnet core, which has its own instrumentation APIs for logging, metrics and tracing. Therefore, OTel collects telemetry from the build in dotnet core APIs. The benefit that OTel brings as an industry standard is a common mechanism for collecting telemetry data and integrating APMs. For more information [see](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel).

:fire: If you are using a different programming language, please check the [supported programming languages](https://opentelemetry.io/docs/languages/).  

### Prometheus server

As illustrated in the figure above, Prometheus is configured with the metrics endpoints of the Web Server and REST API. Extend this configuration if you'd like to inlude other systems/microservices/etc.

#### Install Prometheus server

```bash
ansible-playbook prometheus.yml -i hosts -K -vvv
```

### Collector

For a more flexible setup, you may be interested in the setup of an *Otel* collector. For example `InfluxDB` with [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) and the [OpenTelemetry Input Plugin](https://github.com/influxdata/telegraf/blob/release-1.21/plugins/inputs/opentelemetry/README.md):

```mermaid
flowchart TD;    
    WebServer[Web Server]-- "OTLP GRPC" -->Telegraf;
    RESTAPI[REST API]-- "OTLP GRPC" -->Telegraf;
    Telegraf---->InfluxDB;
    Telegraf---->Kapacitor;
    Kapacitor<---->Chronograf
    Kapacitor<---->InfluxDB;
    InfluxDB---->Grafana;
    InfluxDB---->Zipkin/Prometheus/...;
    InfluxDB<---->Chronograf;
```

For completeness, the diagram above mentions [Chronograf](https://www.influxdata.com/time-series-platform/chronograf/) the user interface and administrative comonent of *InfluxDB* which allows to interact with the [TICK-stack](https://www.influxdata.com/time-series-platform/) and [Zipkin](https://zipkin.io/) a distributed tracing system.  

You can also use the [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/).  

#### Install InfluxDB

```bash
ansible-playbook otelcollector.yml -i hosts -K -vvv
```

After installing, open the UI on `http://{{ groups['otlp_controller'][0] }}:8086)`.

### Grafana server

#### Install Grafana server

```bash
ansible-playbook grafana.yml -i hosts -K -vvv
```

After installing *Grafana*, you can browse to the *Grafana* endpoint (check the *Grafana* endpoint in [hosts](./playbooks/hosts)) and open the `ASP .NET Core` dashboard: 

![ASP .NET Core dashboard](./ASP.NET%20Core%20metrics.png)

## Certificate Authority

Setting up a CA chain with Ansible and automating the process of generating CSRs (Certificate Signing Requests) for each virtual machine/container is a robust and secure way to manage certificates in our infrastructure. For developers, it is probably sufficient to generate certificates used to encrypt internal traffic and for your *public* endpoint in your LAN. If you'd like to publish your *public* endpoint on the Internet, you should use a CA like **Let's Encrypt**.  

1. **Ansible Roles**:
    * **CA Role**: This role sets up a Certificate Authority (CA).
    * **Client Role**: This role will generate CSRs for each virtual machine and handle the signing of certificates by the CA.

2. **CA Role**:
    * Configure the CA server: Install and configure software like OpenSSL to act as the CA.
    * Generate the CA certificate and private key.
    * Create necessary directories to store certificates and keys.
    * Set up appropriate permissions to ensure security.

3. **Client Role**:
    * Install necessary tools like OpenSSL for generating CSRs.
    * Generate CSRs for each virtual machine.
    * Securely transmit the CSRs to the CA server.
    * Sign the CSRs.
    * Retrieve signed certificates from the CA.

By following the steps above, we can set up our own (small/simple) CA chain with Ansible and automate the process of generating CSRs and signing certificates for our virtual machines/internal traffic.

To ensure that consumers trust our client certificates and/or to establish secure communication channels within our infrastructure (network communication within the datacentre), we have several options:

* **Distribute CA Certificate**: Distribute the CA certificate (which was used to sign the client certificates) to the consumers of the client certificates. Consumers can then import the CA certificate into their trust store (e.g., browser's certificate store, operating system's certificate store) as a trusted root CA. This allows the consumers to trust any certificates signed by that CA. :cupid: If you run the Ansible playbooks on Ubuntu, the `caclient` playbook will install the root certificate for you, on Windows, you will have to install the certificate manually (at least, for now).
* **Certificate Chain**: Along with the client certificate, provide the entire certificate chain up to the root CA certificate. This includes the client certificate, any intermediate CA certificates, and the root CA certificate. Consumers can then verify the certificate chain, ensuring that each certificate in the chain is signed by the preceding one, up to the trusted root CA certificate. :fire: In this particular implementation, we have no certificate chain.
* **Publicly Trusted CA**: If you require wider trust, consider obtaining client certificates from a publicly trusted CA (e.g., Let's Encrypt, DigiCert). Certificates issued by publicly trusted CAs are automatically trusted by most systems, as they are included in the trust stores of popular browsers and operating systems. :fire: For our public endpoint, we will use Let's Encrypt, internal traffic, however, will be encrypted with the self-generated client certificates.
* **Custom Trust Store:** Create a custom trust store containing the CA certificate or certificate chain and distribute it to the consumers of the client certificates. Applications can then be configured to use this custom trust store to verify the authenticity of client certificates. :fire: Might be used in a future implementation.
* **Certificate Pinning**: Implement certificate pinning, where consumers explicitly trust a specific client certificate or CA certificate by embedding its fingerprint or public key within the application code. This ensures that only the specified certificate or CA is trusted, providing an additional layer of security. :fire: Might be used in a future implementation.
* **Education and Documentation**: Educate consumers about the trust model and security practices involved in certificate authentication. Provide clear documentation on how to verify and trust client certificates, including instructions for importing CA certificates or certificate chains into trust stores. :fire: Might be used in a future implementation.

In our implementation, the **Client Role** will copy the CA certificate to the default *Ubuntu* certificate stores.

### Install Certificate Authority

```shell
ansible-playbook ca.yml -i hosts -K -vvv
```

### Generate Client Certificates

```shell
ansible-playbook caclient.yml -i hosts -K -vvv
```

## HTTPS Offloading

In the previous section, we explained how certificates are generated with our own CA. We can now use the certificates to encrypt internal traffic after offloading HTTPS traffic.

As we'd like to use HTTPS offloading and load balance the web server and REST endpoint, we migth choose a setup similar to:

```mermaid
graph TB
    client1(Internet Client 1) -->|HTTPS request| nginx[Load Balancing<br/>HTTPS offloading]
    client2(Internet Client 2) -->|HTTPS request| nginx
    client3(Internet Client 3) -->|HTTPS request| nginx

    nginx -->|Distribute request| ws1[Web Server 1]
    nginx -->|Distribute request| ws2[Web Server 2]
    nginx -->|Distribute request| re1[REST Endpoint 1]
    nginx -->|Distribute request| re2[REST Endpoint 2]

    ws1 -->|Optional state management| redis[(REDIS Instance)]
    ws2 -->|Optional state management| redis
    re1 -->|Optional state management| redis
    re2 -->|Optional state management| redis
```

Note that internal traffic (for example the Web Server querying the REST endpoint) runs over a publically acessible *proxy* and is not necessarily encrypted. Moreover, if the proxy acting as the HTTPS offloader and load balancer is compromised, an attacker could potentially gain access to both the unencrypted internal traffic routed through it and the encrypted traffic it decrypts, which makes it a critical point of security.  

Implementing a Demilitarized Zone (DMZ) is an approach to solve this: mitigate risks by segregating different parts of a network, hence, limiting the potential impact of a breach. A potential approach:  

```mermaid
graph TD
    subgraph Internet
      client[Client]
    end
    
    subgraph DMZ
      nginx_public[ProxyPublic]
    end

    subgraph Internal Network
      nginx_internal[ProxyInternal]
      rest[REST Endpoints]
      web[Web Servers]
      redis[(REDIS)]
    end

    client --> nginx_public
    nginx_public -.->|Secure Channel| nginx_internal
    nginx_internal --> rest
    nginx_internal --> web
    rest -->|State Management| redis
    web -->|State Management| redis
```

From the diagram above, the segregation of both networks should be obvious: internal traffic is routed over `ProxyInternal` and not visible on (a breached) `ProxyPublic`.  

As the proxy acts as the entry point for incoming traffic, it can offload SSL/TLS for incoming HTTPS requests. The proxy decrypts the HTTPS traffic, re-encrypts the traffic and then routes the requests to the appropriate backend services. Backend services can - for instance - use certificates from an internal Certificate Authority (CA).  

```mermaid
graph TD
    subgraph Internet
      client[Client]
    end
    
    subgraph DMZ
      nginx_public[ProxyPublic]
    end
    
    subgraph Internal Network
      nginx_internal[ProxyInternal]
      rest[REST Endpoints]
      web[Web Servers]
      redis[(REDIS)]
    end

    client -->|HTTPS| nginx_public    
    nginx_public -->|HTTPS| nginx_internal
    nginx_internal -->|HTTPS| rest
    nginx_internal -->|HTTPS| web
    rest -->|State Management| redis
    web -->|State Management| redis        
```

:fire: TODO to configure a DMZ (firewall), we need some extra info:
| Source IP | Destination IP | Protocol (HTTPS TCP/443 or DNS UDP/53 or ...) |
|:---------:|:--------------:|:-----------------------------------------:|
| 192.168.51|192.168.1.60    |HTTPS TCP/443                              |

### NGINX

The proxy above may be implemented with different solutions:

* NGINX
* Traefik
* HAProxy
* Apache HTTP Server
* IIS
* Envoy Proxy
* Caddy
* F5
* ...

For this paricular setup, we will use NGINX.  

The following NGINX configuration demonstrates a setup designed to facilitate secure HTTPS connections to a load-balanced backend comprised of multiple HTTPS API servers:

```mermaid
flowchart TD;        
    RESTAPI1[REST Endpoint node1];
    RESTAPI2[REST Endpoint node2];
    RESTAPI3[REST Endpoint node2];

    NGINX--load balances/HTTPS-->RESTAPI1;
    NGINX--load balances/HTTPS-->RESTAPI2;
    NGINX--load balances/HTTPS-->RESTAPI3;

    RESTAPI1--data protection-->Redis;
    RESTAPI2--data protection-->Redis;
    RESTAPI3--data protection-->Redis;

    Client--HTTPS-->NGINX
```

The `NGINX` configuration for the following setup illustrates a setup for managing encrypted, load-balanced connections to a backend API cluster, with a focus on detailed logging, security with SSL, and flexibility to support large headers and optional WebSocket connections:  

```nginx
http {
    log_format upstreamlog '[$time_local] $remote_addr - $remote_user - $server_name $host to: $upstream_addr: $request $status upstream_response_time $upstream_response_time msec $msec request_time $request_time';

    upstream https_api {
        server 192.168.1.64:5000;
        server 192.168.1.68:5000;
        server 192.168.1.69:5000;
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        client_header_buffer_size 4k;
        large_client_header_buffers 4 8k; # Increase if JWT or other headers are large      

        access_log /var/log/nginx/access.log upstreamlog;

        # The location ^~ /api/ block captures all requests starting with /api/ and forwards them to the https_api upstream. The ^~ modifier gives this location precedence over regular expression locations that might also match.
        location ^~ /api/ {
            # The proxy_pass https://https_api; directive tells NGINX to forward the requests to the defined upstream group https_api. Ensure the scheme (https://) matches your upstream configuration. If your upstream servers are configured with HTTP, change the scheme accordingly.
            proxy_pass https://https_api;

            # Header adjustments are defined inside the /api/ location block to ensure they're applied only to the proxied requests.          
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Authorization $http_authorization;

            # Additional necessary proxy settings
        }

        # The location / block is modified to return a 404 error for any requests that don't match the /api/ path. This effectively drops traffic not intended for the API.        
        location / {
            return 404;
        }

      # Optional: Redirect the root path specifically if necessary
      # location = / {
      #     return 404;
      # }
        
      # Other possible headers you might want to set
      # proxy_set_header Authorization ""; # If you need to reset the Authorization header

      # WebSocket support (if needed)
      # proxy_http_version 1.1;
      # proxy_set_header Upgrade $http_upgrade;
      # proxy_set_header Connection "upgrade";
    }
}
```

If we'd like to include the web servers in the diagram and configured as above, we get a diagram like:

```mermaid
flowchart TD;        
    subgraph /
        web1[Web Server node1]
        web2[Web Server node2]    
    end
    subgraph /api
        RESTAPI1[REST Endpoint node1];
        RESTAPI2[REST Endpoint node2];
        RESTAPI3[REST Endpoint node2];
    end

    NGINX--load balances/HTTPS-->RESTAPI1;
    NGINX--load balances/HTTPS-->RESTAPI2;
    NGINX--load balances/HTTPS-->RESTAPI3;

    
    NGINX--load balances/HTTPS-->web1;
    NGINX--load balances/HTTPS-->web2;    

    
    web1--data protection-->Redis;
    web2--data protection-->Redis;   

    RESTAPI1--data protection-->Redis;
    RESTAPI2--data protection-->Redis;
    RESTAPI3--data protection-->Redis;

    Client--HTTPS-->NGINX
```

and an NGINX config:

```nginx
http {
    log_format upstreamlog '[$time_local] $remote_addr - $remote_user - $server_name $host to: $upstream_addr: $request $status upstream_response_time $upstream_response_time msec $msec request_time $request_time';

    upstream https_api {
        server 192.168.1.64:5000;
        server 192.168.1.68:5000;
        server 192.168.1.69:5000;
    }

    upstream web_api {
        server 192.168.1.60:5000;
        server 192.168.1.61:5000;
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        # The server_name directive specifies cars.be as the domain this server block will respond to, also for /api/!
        server_name cars.be;

        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        client_header_buffer_size 4k;
        large_client_header_buffers 4 8k; # Increase if JWT or other headers are large      

        access_log /var/log/nginx/access.log upstreamlog;

        # Serve the website for cars.be except for paths under /api/
        location / {
            proxy_pass https://web_api/
        }

        # The location ^~ /api/ block captures all requests starting with /api/ and forwards them to the https_api upstream. The ^~ modifier gives this location precedence over regular expression locations that might also match.
        location ^~ /api/ {
            # The proxy_pass https://https_api; directive tells NGINX to forward the requests to the defined upstream group https_api. Ensure the scheme (https://) matches your upstream configuration. If your upstream servers are configured with HTTP, change the scheme accordingly.
            proxy_pass https://https_api;

            # Header adjustments are defined inside the /api/ location block to ensure they're applied only to the proxied requests.          
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Authorization $http_authorization;

            # Additional necessary proxy settings
        }

        # The location / block is modified to return a 404 error for any requests that don't match the /api/ path. This effectively drops traffic not intended for the API.        
        location / {
            return 404;
        }

      # Optional: Redirect the root path specifically if necessary
      # location = / {
      #     return 404;
      # }
        
      # Other possible headers you might want to set
      # proxy_set_header Authorization ""; # If you need to reset the Authorization header

      # WebSocket support (if needed)
      # proxy_http_version 1.1;
      # proxy_set_header Upgrade $http_upgrade;
      # proxy_set_header Connection "upgrade";
    }
}
```

The configuration above now serves a website for requests to cars.be on port 443 with HTTPS, except for HTTPS requests to cars.be/api/, which are proxied to the upstream https_api.  

As mentioned, the public certificate may be delivered by Let's Encrypt. If we'd like to automatically request/renew certificates, Let's Encrypt will need to check that we are the owners of our domain (cars.be in this example). Let's Encrypt executes such a test by requesting a special file from the http://domain/.well-known/acme-challenge directory. `certbot` Is the most common client for Let's Enrypt to *generate this special file*.  

```nginx
http {
    log_format upstreamlog '[$time_local] $remote_addr - $remote_user - $server_name $host to: $upstream_addr: $request $status upstream_response_time $upstream_response_time msec $msec request_time $request_time';

    upstream https_api {
        server 192.168.1.64:5000;
        server 192.168.1.68:5000;
        server 192.168.1.69:5000;
    }

    upstream web_api {
        server 192.168.1.60:5000;
        server 192.168.1.61:5000;
    }

    server {
        listen 80;
        server_name cars.be;

        location /.well-known/acme-challenge/ {
            root /var/www/letsencrypt;
            allow all;
        }

        location / {
            return 301 https://$host$request_uri;
        }
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        # The server_name directive specifies cars.be as the domain this server block will respond to, also for /api/!
        server_name cars.be;

        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        client_header_buffer_size 4k;
        large_client_header_buffers 4 8k; # Increase if JWT or other headers are large      

        access_log /var/log/nginx/access.log upstreamlog;

        # Serve the website for cars.be except for paths under /api/
        location / {
            proxy_pass https://web_api/;
        }

        # The location ^~ /api/ block captures all requests starting with /api/ and forwards them to the https_api upstream. The ^~ modifier gives this location precedence over regular expression locations that might also match.
        location ^~ /api/ {
            # The proxy_pass https://https_api; directive tells NGINX to forward the requests to the defined upstream group https_api. Ensure the scheme (https://) matches your upstream configuration. If your upstream servers are configured with HTTP, change the scheme accordingly.
            proxy_pass https://https_api;

            # Header adjustments are defined inside the /api/ location block to ensure they're applied only to the proxied requests.          
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Authorization $http_authorization;

            # Additional necessary proxy settings
        }
        
      # Other possible headers you might want to set
      # proxy_set_header Authorization ""; # If you need to reset the Authorization header

      # WebSocket support (if needed)
      # proxy_http_version 1.1;
      # proxy_set_header Upgrade $http_upgrade;
      # proxy_set_header Connection "upgrade";
    }
}
```

In the NGINX configuration examples above, the routing of traffic is based on URLs, if we route traffic based on server names we get the following representation:

```mermaid
flowchart TB
    client([Client]) -->|HTTPS request| nginx[NGINX]
    nginx -->|Decides based on identity.cars.be| identity[NGINX server block </br> for Authentication]
    nginx -->|Decides based on abpapi.cars.be| abpapi[NGINX server block </br> for REST]
    nginx -->|Decides based on cars.be| web[NGINX server block </br> for Web]

    subgraph identityZ [ ]
        identity --> identity_api([Upstream identity_api])
        identity_api -->|HTTPS traffic| Authentication1
        identity_api -->|HTTPS traffic| Authentication2
    end

    subgraph abpapiZ [ ]
        abpapi --> abp_api([Upstream abp_api])
        abp_api -->|HTTPS traffic| REST1
        abp_api -->|HTTPS traffic| REST2
    end

    subgraph webZ [ ]
        web --> web_api([Upstream web_api])
        web_api -->|HTTPS traffic| WEB1
        web_api -->|HTTPS traffic| WEB2
    end    
```

For our particular setup, this becomes:

```nginx
events {
}

http {

    client_header_buffer_size 4k;
    #large_client_header_buffers 4 8k; # Increase if JWT or other headers are large      
    proxy_buffer_size   128k;
    proxy_buffers   4 256k;
    proxy_busy_buffers_size   256k;
    large_client_header_buffers 4 16k;

    log_format upstreamlog '[$time_local] $remote_addr - $remote_user - $server_name $host to: $upstream_addr: $request $status upstream_response_time $upstream_response_time msec $msec request_time $request_time';

    upstream abp_api {
        {% for host in groups['http_api'] %}
        server {{ host }}:{{ hostvars[host]['service_port'] }};
        {% endfor %}
    }

    upstream web_api {        
        {% for host in groups['webserver'] %}
        server {{ host }}:{{ hostvars[host]['service_port'] }};
        {% endfor %}
    }

    upstream identity_api {        
        {% for host in groups['identity_server'] %}
        server {{ host }}:{{ hostvars[host]['service_port'] }};
        {% endfor %}
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        # The server_name directive specifies cars.be as the domain this server block will respond to, also for /api/!
        server_name identity.{{ project_domain }};

        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        access_log /var/log/nginx/access.log upstreamlog;

        # Serve the website for identity.cars.be
        location / {
            proxy_pass https://identity_api/;

            # Header adjustments are defined inside the /api/ location block to ensure they're applied only to the proxied requests.          
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Authorization $http_authorization;

            # Additional necessary proxy settings
        }    
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        # The server_name directive specifies cars.be as the domain this server block will respond to, also for /api/!
        server_name abpapi.{{ project_domain }};

        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        access_log /var/log/nginx/access.log upstreamlog;

        # Serve the website for identity.cars.be
        location / {
            proxy_pass https://abp_api/;

            # Header adjustments are defined inside the /api/ location block to ensure they're applied only to the proxied requests.          
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Authorization $http_authorization;

            # Additional necessary proxy settings
        }    
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        # The server_name directive specifies cars.be as the domain this server block will respond to, also for /api/!
        server_name {{ project_domain }};

        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;

        access_log /var/log/nginx/access.log upstreamlog;

        # Serve the website for cars.be except for paths under /api/
        location / {
            proxy_pass https://web_api/;

            # Header adjustments are defined inside the /api/ location block to ensure they're applied only to the proxied requests.          
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Authorization $http_authorization;

            # Additional necessary proxy settings
        }        
        
      # Other possible headers you might want to set
      # proxy_set_header Authorization ""; # If you need to reset the Authorization header

      # WebSocket support (if needed)
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
    }
}
```

As routing based on hostnames has less impact on the web server and REST endpoint source code (no constraints on URLS), routing based on hostnames is the preferred solution in this project.  

#### Log format

To capture detailed information about requests and responses, we define a custom logging format: the `log_format` directive specifies a custom format named `upstreamlog`, which includes *timestamps, client IP, the requested server name and host, the upstream server address that handled the request, the request details, status code, response times, and request processing times*. This detailed logging is crucial for debugging and monitoring the performance of the upstream servers, in particular inspecting load balancing as the log will display the IP address of the upstream REST API node.

#### Load balancing

Distribute incoming traffic across multiple backend servers: the upstream block named `http_api` defines a cluster of servers (with specified IP addresses and port numbers) that requests will be load balanced across. This setup increases the application's scalability and reliability by distributing the load and providing redundancy.

#### HTTPS configuration

We configure NGINX to serve HTTPS traffic and to use specific SSL parameters and certificates.

* The **listen 443 ssl directives** instruct NGINX to listen for incoming connections on port 443 with SSL encryption, including IPv6 connections ([::]:443 ssl).
* The **include directives** incorporate additional SSL configurations and self-signed certificate details from external files (self-signed.conf and ssl-params.conf), modularizing the SSL setup for easier management.
* client_header_buffer_size And large_client_header_buffers directives adjust the buffer sizes for client request headers, accommodating potentially large JWT tokens or other large headers.

Note the HTTPS offloading and re-encryption in the configuration above which involves the NGINX server acting as a termination point for incoming HTTPS connections from clients. NGINX decrypts these connections, inspects the traffic, and then re-encrypts the traffic before forwarding it to the backend servers over HTTPS using separate, internal SSL certificates. This process allows for secure communication both externally with clients and internally within the data center:

* **Decryption of Incoming HTTPS Requests**: NGINX uses public-facing SSL certificates to decrypt data received over SSL/TLS from clients. This is the offloading part.
* **Inspection and Processing**: Once decrypted, NGINX can inspect, log, or manipulate the HTTP content as necessary. This step can involve modifying headers, applying access controls, or making routing decisions.
* **Re-Encryption and Forwarding**: NGINX then re-encrypts the requests using internal SSL certificates and forwards them to the backend servers over HTTPS. This ensures that traffic remains secure as it traverses the internal network.
* **Secure Internal Communication**: The backend servers, configured with the internal certificates, decrypt the re-encrypted requests, process them, and send back the responses. NGINX then encrypts the responses again (if necessary) before sending them back to the clients.

HTTPS offloading and re-encryption has the following benefits:

* **Enhanced Security Across the Board**: This setup ensures end-to-end encryption of data, maintaining security from the client to the NGINX server and from the NGINX server to the backend servers. Using internal certificates for the data center adds an extra layer of security within the internal network.
* **Centralized Public SSL Management**: Public-facing SSL certificate management remains centralized at the NGINX server, simplifying the administration of public encryption keys and certificates.
* **Flexible Trust and Security Policies**: The separation of external and internal certificates allows for distinct trust domains and security policies. For example, stronger or different encryption standards can be applied internally compared to what is used for public internet traffic.
* **Inspection and Added Controls**: Decrypting the traffic at the NGINX layer allows for detailed inspection and the application of additional security controls before re-encrypting and sending it to backend services. This can be crucial for compliance with security policies, logging, or performing deep packet inspection for threat detection.
* **Performance and Scalability**: While the NGINX server handles the computational overhead of encrypting and decrypting traffic twice, it offloads backend servers from doing so. This can be optimized by using hardware that accelerates SSL processing. Furthermore, it allows backend servers to be scaled out without the need to manage public SSL certificates on each of them.
* **Secure Internal Traffic**: Using HTTPS internally protects against potential threats lurking within the data center, ensuring that sensitive data is encrypted in transit even within the network's trusted boundaries.

In summary, employing HTTPS offloading and re-encryption with NGINX provides robust security by ensuring encrypted traffic both externally and internally while centralizing certificate management and offering the ability to inspect and manipulate HTTP traffic. This configuration enhances the overall security posture and flexibility of managing traffic flows within distributed application architectures.

#### Access logging

In orde to use a custom-defined log file path and format, the `access_log` directive specifies the path to the access log file and instructs NGINX to use the *upstreamlog* format (`log_format upstreamlog '[$ti...`) for logging, ensuring that detailed information about each request and its handling is logged for future analysis.

#### Location block and proxy settings

Request routing and proxying behavior for the root path is defined with:

* The **location / block** matches all requests and uses the proxy_pass directive to forward them to the http_api upstream cluster, enabling load balancing.
* The **proxy_set_header directives** modify request headers to include the original host, the real client IP address, and the protocol used, ensuring that the proxied requests contain necessary information for backend services to process them correctly.
* The configuration optionally supports **forwarding the Authorization header**, preserving JWT or other authentication tokens needed by the backend services.

#### Optional websocket support

This part of the configuration provides the necessary headers to support WebSocket connections if needed:

* The commented-out directives (`proxy_http_version`, `proxy_set_header Upgrade`, and `proxy_set_header Connection`) are set up to enable WebSocket support, ensuring that NGINX can proxy WebSocket connections correctly. These lines can be uncommented and adjusted as needed based on the application's requirements.

## References

* [Playbooks directory](https://charlesreid1.com/wiki/Ansible/Directory_Layout/Details)
* [Mardown all-in-one](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)

## TODO

* move proxmox playbook to a different "folder"
* align the [vars](./playbooks/proxmox.cars.be/vars/main.yml) file with an *Ansible* `inventory`
* generate the *Ansible* `inventory` file or use an `inventory` file as a *vars.yml* file to setup *VM*s (if possible)
* integrate [prometheus alerts](https://github.com/prometheus/alertmanager) or [grafana alerts](https://grafana.com/docs/grafana/latest/alerting/fundamentals/alertmanager/) or ...
* [ssh fingerprint checks](https://stackoverflow.com/questions/32297456/how-to-ignore-ansible-ssh-authenticity-checking)
* [enable tcpdump in a container](https://cloud.garr.it/support/kb/general/enableTcpdumpInLXCContainer/)
https://netsplit.uk/posts/2022/10/19/remote_ovh_lab/
* pfSense `pfctl -d` and `pfctl -e` to disable/enable the packet filter
* add DNS
* remove cert generation outside of ca/caclient roles
