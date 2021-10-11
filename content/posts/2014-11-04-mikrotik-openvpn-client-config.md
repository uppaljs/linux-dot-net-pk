---
title: Mikrotik OpenVPN Client Config
author: uppal
type: post
date: 2014-11-04T12:24:12+00:00
url: /mikrotik-openvpn-client-config/
spacious_page_layout:
  - default_layout
categories:
  - Howtos
  - mikrotik

---
Following up on my previous post , Below is the configuration for setting up an OPENVPN Client on Mikrotik Router

<pre class="qoate-code">/interface openvpn-client add name="ovpn-out1" mac-address= max-mtu=1350 
      connect-to=SERVER_IP port=1194 mode=ip user="USERNAME" 
      password="PASSWORD" profile=default certificate=none auth=sha1 
      cipher=blowfish128 add-default-route=no 
</pre>

After this goto **IP > Firewall > NAT** and add a **SRC nat** rule to masqurade all traffic going towards the VPN

<pre class="qoate-code">/ip firewall nat add chain=srcnat action=masquerade out-interface=ovpn-out1 log=no log-prefix=""
</pre>

oh and make sure you have a static route added for your VPN server IP address towards your primary gateway.

and that should be it!

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->