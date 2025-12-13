---
title: "Reverse SSH Tunnel"
date: 2008-07-21
categories:
  - Howtos
tags:
  - ssh
  - linux
  - networking
draft: false
---

ISPs in PK don't generally allow incoming traffic to your public IP rendering it useless, this is a real pain when you want to access your machine from a remote location. The solution to it is a reverse ssh tunnel, which will work in the following way:

**[Home Machine] ==> {Middle Machine} ==> {You}**

So you'll map a port from your home machine to your middle machine, which when connected to from the middle machine will forward all traffic to your home machine. I'll give an example on how to achieve it.

On home machine, do this:

```bash
user@home$ ssh -N -f -R 2222:localhost:22 user@middle-machine
```

Enter the password for your middle machine and that's it.

Now you can connect to the middle machine from anywhere in the world and do:

```bash
user@middle-machine$ ssh user@localhost -p 2222
```

and it will forward it to the ssh server running on your home machine.

Oh btw, you'll need to change the following in `/etc/ssh/sshd_config`:

```
TCPKeepAlive yes
ClientAliveInterval 30
ClientAliveCountMax 99999
```

Have fun!
