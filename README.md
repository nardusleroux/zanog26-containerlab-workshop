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
> This allows to access your VM with `ssh user@<vmID>.<example.com> 


A Visual Studio Code web interface running on HTTPS port 8443. Optionally use your own VS Code installation on your laptop, to connect remotely to the instance.

At the end of each activity, a solution proposal is provided. This allows you to become unstuck or to continue with next activity since activities build on previous activities.

## 1. Containerlab installation

## 2. Creating your first containerlab topology

## 3. Running & configuring your lab

## 4. Adding startup configuration

## 5. Extending your topology

## 6. VSCode plugin

## 7. Traffic capture & link impairment



