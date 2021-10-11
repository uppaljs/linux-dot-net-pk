---
title: Reverse SSH Tunnel
author: uppal
type: post
date: 2008-07-21T18:07:58+00:00
url: /reverse-ssh-tunnel/
categories:
  - Uncategorized

---
ISPs in PK don&#8217;t generally allow incoming traffic to your public IP rendering it useless, this is a real pain , when you want to access your machine from a remote location. The solution to it is a reverse ssh tunnel, which will work in the following way

**[Home Machine] ==> {Middle Machine} ==> {You}**

So you&#8217;ll map a port from your home machine to your middle machine , which when connected to from the middle machine will forward all traffic to your home machine. I&#8217;ll give an example on howto acheieve it.

On home machine , do this

**user@home$ ssh -N -f -R 2222:localhost:22 user@middle-machine**

Enter the password for your middle machine and that&#8217;s it.

Now you can connect to the middle machine from any where in the world and do

**user@middle-machine$ ssh user@localhost -p 2222**

and it will forward it to the ssh server running on your home machine.

Oh btw, you&#8217;ll need to change the following in **/etc/ssh/sshd_config**

**TCPKeepAlive yes  
ClientAliveInterval 30  
ClientAliveCountMax 99999**

Have fun 🙂

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->