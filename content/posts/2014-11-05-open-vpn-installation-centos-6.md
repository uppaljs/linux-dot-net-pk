---
title: Open VPN Installation on Centos 6.5
author: uppal
type: post
date: 2014-11-05T18:17:00+00:00
url: /open-vpn-installation-centos-6/
spacious_page_layout:
  - default_layout
categories:
  - DevOps
  - Howtos

---
In a rush , so will jot down the steps only. You&#8217;ll need epel repo

On CentOS 6 , that will be

<pre class="qoate-code">wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
sudo rpm -Uvh epel-release-6*.rpm
</pre>

Once that is done , 

1. yum install openvpn easy-rsa  
2. mkdir -p /etc/openvpn/easy-rsa/keys  
3. cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa/  
4. vi /etc/openvpn/easy-rsa/vars

Modify these parameters to suit your need

<pre class="qoate-code">[...]
# Don't leave any of these fields blank.
export KEY_COUNTRY="PK"
export KEY_PROVINCE="Punjab"
export KEY_CITY="Multan"
export KEY_ORG="LinuxPakistan"
export KEY_EMAIL="vpn@linux.net.pk"
export KEY_OU="server"
[...]
</pre>

5. cd /etc/openvpn/easy-rsa/  
6. cp openssl-1.0.0.cnf openssl.cnf  
7. source ./vars  
8. ./clean-all  
9. ./build-ca

<pre class="qoate-code">Generating a 2048 bit RSA private key
......................................................+++
............................................................+++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [PK]: ----> Press Enter
State or Province Name (full name) [Punjab]: ----> Press Enter
Locality Name (eg, city) [Multan]: ----> Press Enter
Organization Name (eg, company) [LinuxPakistan]: ----> Press Enter
Organizational Unit Name (eg, section) [server]: ----> Press Enter
Common Name (eg, your name or your server's hostname) [server]: ----> Press Enter
Name [EasyRSA]: ----> Press Enter
Email Address [vpn@linux.net.pk]: ----> Press Enter
</pre>

10. ./build-key-server server  
11. ./build-key client ( if you want to use RSA Keys for Connectivity )

12. ./build-dh  
13. cd /etc/openvpn/easy-rsa/keys/  
14. cp dh2048.pem ca.crt server.crt server.key /etc/openvpn/  
15. vi /etc/openvpn/server.conf

Use my configuration file from <a href="http://linux.net.pk/blog/openvpn-configuration/" title="OpenVPN Configuration" target="_blank">this post</a> 

And that should do it

**  
IP Routing**

<pre class="qoate-code">vi /etc/sysctl.conf
</pre>

Set the value of the parameter below to 1 to allow IP Packet Forwarding from VPN clients

<pre class="qoate-code"># Controls IP packet forwarding
net.ipv4.ip_forward = 1
</pre>

Reload sysctl

<pre class="qoate-code">sysctl -p
</pre>

Finally some iptables magic to do the masquerading

<pre class="qoate-code">iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
</pre>

Save and Restart the Firewall

<pre class="qoate-code">service iptables save
service iptables restart
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->