# ZANOG26 Containerlab workshop
Containerlab introduction material for [ZANOG26](https://iweek.org.za/) pre event workshop.

## 0.1 Getting connected
You should have received a note with Wi-Fi connectivity details, your instance ID and credentials on it.

Your VM can be reached on `<vmID>.<example.com>`.  

SSH access is via a Jumphost.
Access your VM with any OpenSSH client with: 
```
ssh -J user@<example.com:2222> user@<vmID>
```

> [!TIP]
> For simplified access in your SSH config file (~/.ssh/config), you can set it up like this:  
> ```
> Host jump-host
>     HostName <example.com>
>     User jump-user
>
> Host <vmID>
>     HostName 192.168.50.<vmID>
>     User user
>     ProxyJump jump-host
>```
> This allows to access your VM with `ssh user@<vmID>.<example.com>`


A Visual Studio Code web interface running on HTTPS port 8443. Optionally use your own VS Code installation on your laptop, to connect remotely to the instance.

At the end of each activity, a solution proposal is provided. This allows you to become unstuck or to continue with next activity since activities build on previous activities.

## 1. Containerlab installation

### Task 1.1 Install Containerlab
First order of business is to get [Containerlab installed](https://containerlab.dev/install/). Containerlab requires the following pre-requisites:
* A Linux user with sudo priviledges
* A working Docker engine installation
* NOS Container images (e.g. Nokia SR Linux, VPP) that are not downloadable from a container registry.

Containerlab provides a [quick-start script](https://containerlab.dev/quickstart/) that works with most DEB (Debian, Ubuntu) and RPM-based (Red Hat, CentOS, Fedora, etc) distributions and will take care of Docker install as well.

After logging in to your VM, run the following command:

```
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

This one liner will install Docker and Containerlab.

> [!TIP]
> If you would like to use Containerlab without sudo, like the output of the installer suggests, you should add your user to this group using the following command:  
> 
> `gpasswd -a <your username here> clab_admins`
> 
> and apply the new group membership to your terminal session using the command
> `newgrp clab_admins` (or log in and out from the SSH session)

The Containerlab installation can be tested by running `containerlab version`:
```
$ clab version

 ⣴⡾⠛⠛⠖ ⢠⣶⠟⠛⢷⣦ ⢸⣿⣧  ⣿ ⠘⠛⢻⡟⠛⠛  ⣾⣿⡀  ⣿⡇ ⣿⣿⡄ ⢸⡇ ⣿⡟⠛⠛⠃⢸⣿⠛⠛⣷⡄ ⣿⡇      ⣿⡇
⢸⣿     ⣿⡇     ⣿⡇⢸⣿⠹⣧ ⣿   ⢸⡇   ⣸⡏⢹⣧  ⣿⡇ ⣿⡏⢿⣄⢸⡇ ⣿⣧⣤⣤ ⢸⣿⣀⣀⣾⠇ ⣿⡇⠐⠟⠛⢿⡆ ⣿⡷⠛⢿⣆
⠘⣿⣄  ⡀ ⢻⣧⡀ ⣠⣿⠃⢸⣿ ⠘⣷⣿   ⢸⡇  ⢠⣿⠷⠶⢿⡆ ⣿⡇ ⣿⡇ ⢻⣾⡇ ⣿⡇    ⢸⣿⠉⢻⣧⡀ ⣿⡇⢰⡟⠛⣻⡇ ⣿⡇ ⣸⡿
 ⠈⠙⠛⠛⠉  ⠉⠛⠛⠋⠁ ⠘⠛  ⠈⠛   ⠘⠃  ⠚⠃  ⠘⠛ ⠛⠃ ⠛⠃  ⠙⠃  ⠛⠛⠛⠛⠃⠘⠛  ⠙⠓  ⠛⠃⠈⠛⠛⠙⠃ ⠛⠛⠛⠛⠁

    version: 0.74.0
     commit: 11553904b
       date: 2026-03-08T12:25:47Z
     source: https://github.com/srl-labs/containerlab
 rel. notes: https://containerlab.dev/rn/0.74/
```

Containerlab runs in most places. Head over to [Containerlab installation](https://containerlab.dev/install/) instructions for [Windows WSL](https://containerlab.dev/install/#windows) or [macOS](https://containerlab.dev/install/#apple-macos). This also contains manual instructions and more.

### Task 1.2: Testing Containerlab

In order to test and validate your Containerlab installation, you can deploy a "hello world" topology example and test if traffic passes between two nodes.

The following command clones the provided Git repository, and deploys the Containerlab topology contained within.

```
clab deploy -t https://github.com/nardusleroux/clab-helloworld
```

Once deployed, you should be able to ping from `client1` to `client2`.

```
$ docker exec clab-helloworld-client1 ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=64 time=1.037 ms
```

Finally, we can clean up the topology we just created by doing

```
clab destroy -t https://github.com/nardusleroux/clab-helloworld
```

**What did we learn?**
- Containerlab has a command alias, `clab`, which is quicker to type.
- Containerlab topologies can be deployed from remote URLs, which is great to share labs.
GitHub repositories, direct URLs to Containerlab topologies, even S3 links work.
- `docker exec` can be used to run commands on containers deployed by Containerlab.
This is one of the two main methods (ssh being the other) for interacting with a node in a running Containerlab topology.

## 2. Creating and running your first Containerlab topology
A valid Containerlab topology consist of a name, and a set of nodes and links between them.

#### Important terminology

The most important objects within a Containerlab topology definition are:

- [Nodes](https://containerlab.dev/manual/nodes/#nodes)
Node object is one of the pillars of Containerlab. Within the `nodes` section of a Containerlab topology definition the actual containers that should be deployed are described. In most cases, each node represents a single container. This could be a network element (router, switch, firewall, etc), but can also be any other Linux-based container.
- [Links](https://containerlab.dev/manual/topo-def-file/#links)
Nodes can be deployed without links, but to create a network topology you would want them. In the `links` section, we can define links by providing pairs of `endpoints` for Containerlab to connect. An endpoint consist of a `node` and an interface of the node.

A [topology definition deep-dive document](https://containerlab.dev/manual/topo-def-file/) provides a complete reference of the topology definition syntax.

#### Task 2.1 - Your first lab topology

Containerlab supports [over 50 different types of Network OSes](https://containerlab.dev/manual/kinds/) that can be ran inside a topology. However, most commercial NOSes can only be downloaded with an active account for a given vendor's website or download portal.

In this workshop, we will use two freely downloadable NOSes that do not require any registration:

- [FD.io VPP](https://fd.io/)
- [Nokia SR Linux](https://learn.srlinux.dev/)

To get started, we will create a simple topology called `zanog26-workshop` consisting of 2 nodes with a single link in between them.

`leaf1` will be an SR Linux node, running SR Linux 25.10, and `leaf2` will be VPP, running the latest version, using the container image `git.ipng.ch/ipng/vpp-containerlab:latest`

To make things simple, we will use the first available data-plane interface for both nodes:  
in case of SR Linux, this is `ethernet-1/1`, and `eth1` for VPP.

![Topology](images/2.topology.png)

Based on the topology diagram and the [Containerlab topology format definition](https://containerlab.dev/manual/topo-def-file/#topology-definition-components), you should be able to define your first Containerlab topology in the file `zanog26-workshop.clab.yaml`

If you are already familiar with the Containerlab basics and want to skip over this exercise, you'll find the solution right here:

<details>
<summary>Task 2.1 solution</summary>

```yaml
name: zanog26-workshop

topology:
  nodes:
    leaf1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:25.10
    leaf2:
      kind: linux
      image: git.ipng.ch/ipng/vpp-containerlab:latest

  links:
    - endpoints: ["leaf1:ethernet-1/1", "leaf2:eth1"]
```

</details>

**What did we learn ?**
- How to define your own topology.

## 3. Running & configuring your lab

#### Task 3.1 - Deploying the topology
Next step would be to deploy the lab and start with network configuration.

First deploy the lab using the `containerlab deploy` command.

> [!TIP]
> If your topology file has the .clab.yml postfix, `containerlab deploy` or `clab deploy` is sufficient to deploy the topology. You can also specifiy the topology file to use by using the `containerlab deploy -t <topologyfile.yml>`

During the deploy process, Containerlab will automatically download the necessary container images if they are not already present on the VM.

Once the nodes have started, Containerlab will give us an overview like this:
```
╭─────────────────────────────┬──────────────────────────────────────────┬─────────┬───────────────────╮
│             Name            │                Kind/Image                │  State  │   IPv4/6 Address  │
├─────────────────────────────┼──────────────────────────────────────────┼─────────┼───────────────────┤
│ clab-zanog26-workshop-leaf1 │ nokia_srlinux                            │ running │ 172.20.20.5       │
│                             │ ghcr.io/nokia/srlinux:25.10              │         │ 3fff:172:20:20::5 │
├─────────────────────────────┼──────────────────────────────────────────┼─────────┼───────────────────┤
│ clab-zanog26-workshop-leaf2 │ linux                                    │ running │ 172.20.20.4       │
│                             │ git.ipng.ch/ipng/vpp-containerlab:latest │         │ 3fff:172:20:20::4 │
╰─────────────────────────────┴──────────────────────────────────────────┴─────────┴───────────────────╯
```
The nodes are automatically assigned a management IPv4 and IPv6 address by Containerlab, this can also be set statically in the topology file. Containerlab will also populate the hosts file to use the Name of the node to connect.

> [!Note]
>
> Given that most network OSes require additional configuration to be remotely manageable, Containerlab > loads them with an initial configuration, just enough to give them a hostname, management connectivity > and fixed credentials to SSH in with.
>
> Some node kinds go a bit further than that - Containerlab will automatically load SSH public keys, pre-configure node hardware configuration, as a convenience feature.

For most nodes, connecting to a node is just an ssh session to the node's name.

> [!Tip]
>
> The default username and password configured for SR Linux deployed via Containerlab is `admin` and `NokiaSrl1!`
> You can find out what default password is assigned to each node kind on the kind's documentation page in the Containerlab Kinds documentation page.

```
user@vm1:~/zanog26-workshop$ ssh admin@clab-zanog26-workshop-leaf1
Warning: Permanently added 'clab-zanog26-workshop-leaf1' (ED25519) to the list of known hosts.
................................................................
:                  Welcome to Nokia SR Linux!                  :
:              Open Network OS for the NetOps era.             :
:                                                              :
:    This is a freely distributed official container image.    :
:                      Use it - Share it                       :
:                                                              :
: Get started: https://learn.srlinux.dev                       :
: Container:   https://go.srlinux.dev/container-image          :
: Docs:        https://doc.srlinux.dev/25-10                   :
: Rel. notes:  https://doc.srlinux.dev/rn25-10-2               :
: YANG:        https://yang.srlinux.dev/v25.10.2               :
: Discord:     https://go.srlinux.dev/discord                  :
: Contact:     https://go.srlinux.dev/contact-sales            :
................................................................

(admin@clab-zanog26-workshop-leaf1) Password:
Last login: Wed Mar 18 11:51:23 2026 from 3fff:172:20:20::1
Loading environment configuration file(s): ['/etc/opt/srlinux/srlinux.rc']
Welcome to the Nokia SR Linux CLI.

--{ running }--[  ]--
A:admin@leaf1#
```

Verify connectivity to leaf2 as well.

> [!Tip]
>
> The default username and password configured for VPP deployed via Containerlab is `root` and `vpp`

```
user@vm1:~/zanog26-workshop$ ssh root@clab-zanog26-workshop-leaf2 
Warning: Permanently added 'clab-zanog26-workshop-leaf2' (ED25519) to the list of known hosts.
root@clab-zanog26-workshop-leaf2's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.12.73+deb13-cloud-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Wed Mar 18 11:52:57 2026 from 3fff:172:20:20::1
root@leaf2:~# vppctl
    _______    _        _   _____  ___ 
 __/ __/ _ \  (_)__    | | / / _ \/ _ \
 _/ _// // / / / _ \   | |/ / ___/ ___/
 /_/ /____(_)_/\___/   |___/_/  /_/    

vpp-clab 
```


#### Task 3.2 - Adding configuration to your topology
Repeat the ping test we did in the "hello-world" lab between leaf1 and leaf2. Configure 10.0.0.1/24 and 10.0.0.2/24 on leaf1 and leaf2 respectively, and perform a ping test.

XXXX

Links to relevant documentations:

FRR
SR Linux
Tip

SR Linux Hints

Navigating the CLI
Configuring an interface
Assigning an interface to a network-instance
Managing the configuration
Tip

FRR tips
✅ Task 1.4 Solution

leaf1 SR Linux configuration
leaf2 FRR configuration
Validation
--{ running }--[  ]--
A:leaf1# ping 10.0.0.2 network-instance default
Using network instance default
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=51.4 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=2.17 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=2.15 ms
^CCommand execution aborted : 'ping 10.0.0.2 network-instance default '

--{ running }--[  ]--


## 4. Adding startup configuration

## 5. Extending your topology

## 6. VSCode plugin

## 7. Traffic capture & link impairment



