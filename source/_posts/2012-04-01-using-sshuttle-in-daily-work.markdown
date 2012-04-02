---
layout: post
title: "Using sshuttle in daily work"
date: 2012-04-01 13:13
comments: true
categories: 
- tools
- tools-network
- sshuttle
---

I was first introduced to [**sshuttle**](https://github.com/apenwarr/sshuttle) by Sooyoung ([@5ooyoung](https://twitter.com/#!/5ooyoung)) in [Favorite Medium](http://favoritemedium.com/) 
as a workaround to The Great Firewall in China.

Since then, it has become my **light-weight network tunneling tool** in daily work. 

Install sshuttle
------
The installation is easy now. You can install it through Mac OSX Homebrew, or [Ubuntu apt-get](http://packages.ubuntu.com/precise/sshuttle).

```bash
brew install sshuttle
```

I use sshuttle to..
------

### 1. Tunnel all traffic 

This is the first command I learned.
It **forwards all TCP traffic and DNS requests to a remote SSH server**.

```bash
sshuttle --dns -vr ssh_server 0/0
```
Just like `ssh`, you can use any server specified in `~/.ssh/config`. 
The `-v` flag means verbose mode. 

Besides TCP and DNS, currently `sshuttle` **does not forward other requests 
_such as_ UDP, ICMP ping etc**.

### 2. Tunnel all traffic, but exclude some 
You can **exclude certain TCP traffic using `-x` option**.

```bash
sshuttle --dns -vr ssh_server -x 121.9.204.0/24 -x 61.135.196.21 0/0 
```

For instance, when I am in China, I don't want to tunnel 
[Youku.com](http://youku.com) traffic to a foreign server,
because its movie streaming service is only available within China. 

In this case, I use `-x` option to exclude Youku.com IP addresses.

### 3. Tunnel only certain traffic

To tunnel only certain TCP traffic, **specify the IP addresses 
or [IP ranges](http://www.subnet-calculator.com/cidr.php) that need tunneling**.

```bash
sshuttle -vr ssh_server 121.9.204.0/24 61.135.196.21 
```

This command comes in handy, whenever I need to test an app feature (e.g. Netflix movie streaming)
which only available in certain countries, or to bypass ISP faulty caches.

### 4. VPN to office network

I seldom do VPN, but all you need is **the remote SSH server with `-NH` flags turned on**. 

```bash
sshuttle -NHvr office_ssh_server
```

`-N` flag tells sshuttle to figure out by itself the IP subnets to forward, 
and `-H` flag to scan for hostnames within remote subnets and store them temporarily in `/etc/hosts`. 


IP addresses.. troublesome?
-----
Well, that's how I feel :) So I have some 
**[sshuttle helpers](https://github.com/teohm/dotfiles/blob/master/.bashrc.d/sshuttle_helpers)
(few lines of Bash functions)** that allow me to **use domain names instead of IP addresses**:

### Tunnel all traffic 
```bash
tnl
```

### Tunnel all traffic, but exclude some 
```bash
tnlbut youku.com weibo.com
```

### Tunnel only certain traffic
```bash
tnlonly netflix.com movies.netflix.com
```

### VPN to office network
```bash
vpnto office_ssh_server
```

The script is available on [my GitHub repo](https://github.com/teohm/dotfiles/blob/master/.bashrc.d/sshuttle_helpers).
You can load it into your `~/.bashrc`. **To override the default tunneling SSH server** in the script:
```
TNL_SERVER=user@another_server tnl
```

