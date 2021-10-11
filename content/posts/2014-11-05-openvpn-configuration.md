---
title: OpenVPN Configuration
author: uppal
type: post
date: 2014-11-04T19:00:26+00:00
url: /openvpn-configuration/
spacious_page_layout:
  - default_layout
categories:
  - Howtos
  - mikrotik
tags:
  - linux
  - mikrotik
  - openvpn

---
Below is the configuration i use for OpenVPN server , It supports user / password authentication,TCP mode and disables TLS which is not supported by Mikrotik ( atleast for now )

<pre class="qoate-code">local IPADDRESS
port 1194
proto tcp
dev tun
tun-mtu 1420
tun-mtu-extra 32
mssfix 1450
ca /etc/openvpn/easy-rsa/2.0/keys/ca.crt
cert /etc/openvpn/easy-rsa/2.0/keys/server.crt
key /etc/openvpn/easy-rsa/2.0/keys/server.key
dh /etc/openvpn/easy-rsa/2.0/keys/dh1024.pem
plugin /usr/share/openvpn/plugin/lib/openvpn-auth-pam.so /etc/pam.d/login
client-cert-not-required
username-as-common-name
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 5 30
#comp-lzo
persist-key
persist-tun
status server-tcp.log
verb 5
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->