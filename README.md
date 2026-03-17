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
⢸⣿     ⣿⡇   ⣿⡇⢸⣿⠹⣧⡀⣿   ⢸⡇   ⣸⡏⢹⣧  ⣿⡇ ⣿⡏⢿⣄⢸⡇ ⣿⣧⣤⣤ ⢸⣿⣀⣀⣾⠇ ⣿⡇⠐⠟⠛⢿⡆ ⣿⡷⠛⢿⣆
⠘⣿⣄  ⡀ ⢻⣧⡀ ⣠⣿⠃⢸⣿ ⠘⣷⣿   ⢸⡇  ⢠⣿⠷⠶⢿⡆ ⣿⡇ ⣿⡇ ⢻⣾⡇ ⣿⡇   ⢸⣿⠉⢻⣧⡀ ⣿⡇⢰⡟⠛⣻⡇ ⣿⡇ ⣸⡿
 ⠈⠙⠛⠛⠉  ⠉⠛⠛⠋⠁ ⠘⠛  ⠈⠛   ⠘⠃  ⠚⠃  ⠘⠛ ⠛⠃ ⠛⠃  ⠙⠃ ⠛⠛⠛⠛⠃⠘⠛  ⠙⠓ ⠛⠃⠈⠛⠛⠙⠃ ⠛⠛⠛⠛⠁

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

**What did we learn from?**
- Containerlab has a command alias, `clab`, which is quicker to type.
- Containerlab topologies can be deployed from remote URLs, which is great to share labs.
GitHub repositories, direct URLs to Containerlab topologies, even S3 links work.
- `docker exec` can be used to run commands on containers deployed by Containerlab.
This is one of the two main methods (ssh being the other) for interacting with a node in a running Containerlab topology.

## 2. Creating your first containerlab topology

## 3. Running & configuring your lab

## 4. Adding startup configuration

## 5. Extending your topology

## 6. VSCode plugin

## 7. Traffic capture & link impairment



