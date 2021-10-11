---
title: Routing Traffic to VPN on Mikrotik
author: uppal
type: post
date: 2014-11-03T23:30:36+00:00
url: /routing-traffic-vpn-mikrotik/
spacious_page_layout:
  - default_layout
categories:
  - Howtos
  - mikrotik

---
I am addicted to spotify and netflix and since I am based out of middle east and use a lot of devices ( hand helds , laptops , media streaming boxes etc. ) , the only solution was to implement routing to my VPN service on the gateway which in my case is a mikrotik RB2011UAS-iNHD.

I also didn&#8217;t want to route all of my traffic to the VPN since it slows down the internet considerably due to latency / headers etc . , so had to come up with selective routing of certain services.

Below is the procedure to do selective routing to the VPN ( I use openvpn in TAP mode and NAT is implemented on the server ). I&#8217;ll use spotify as an example.

&nbsp;

1. Head over to bgp.he.net and search for Spotify to get all the ip addresses associated with spotify

2. Next we will create an address list containing all the addresses in IP > Firewall > Address Lists in mikrotik . We&#8217;ll call this address list OVPN.

3. Then we&#8217;ll create a firewall mangle rule for all traffic going to 0.0.0.0/0 with destination address matching our list OVPN to add a routing mark OVPN-mark

4. We&#8217;ll then create a route on mikrotik that will route all traffic going to the internet with routing mark OVPN-mark to be routed through our VPN link

So here we go

<pre class="qoate-code">/ip firewall address-list
add address=78.31.8.0/22 disabled=no list=OOVPN comment=spotify.com
add address=78.31.12.0/22 disabled=no list=OVPN comment=spotify.com
add address=23.92.96.0/22 disabled=no list=OVPN comment=spotify.com
add address=23.92.104.0/22 disabled=no list=OVPN comment=spotify.com
add address=23.92.100.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.71.232.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.71.148.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.68.28.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.68.183.0/24 disabled=no list=OVPN comment=spotify.com
add address=194.68.181.0/24 disabled=no list=OVPN comment=spotify.com
add address=194.68.176.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.68.169.0/24 disabled=no list=OVPN comment=spotify.com
add address=194.68.116.0/24 disabled=no list=OVPN comment=spotify.com
add address=194.14.177.0/24 disabled=no list=OVPN comment=spotify.com
add address=194.132.196.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.132.176.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.132.162.0/24 disabled=no list=OVPN comment=spotify.com
add address=194.132.152.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.103.36.0/22 disabled=no list=OVPN comment=spotify.com
add address=194.103.13.0/24 disabled=no list=OVPN comment=spotify.com
add address=194.103.10.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.235.51.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.235.32.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.235.232.0/22 disabled=no list=OVPN comment=spotify.com
add address=193.235.224.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.235.206.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.235.203.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.234.240.0/22 disabled=no list=OVPN comment=spotify.com
add address=193.182.8.0/21 disabled=no list=OVPN comment=spotify.com
add address=193.182.7.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.182.3.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.182.244.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.182.243.0/24 disabled=no list=OVPN comment=spotify.com
add address=193.181.4.0/22 disabled=no list=OVPN comment=spotify.com
add address=193.181.184.0/23 disabled=no list=OVPN comment=spotify.com
add address=193.181.180.0/22 disabled=no list=OVPN comment=spotify.com
add address=192.165.160.0/22 disabled=no list=OVPN comment=spotify.com
add address=192.121.53.0/24 disabled=no list=OVPN comment=spotify.com
add address=192.121.140.0/24 disabled=no list=OVPN comment=spotify.com
add address=192.121.132.0/24 disabled=no list=OVPN comment=spotify.com
</pre>

Next We&#8217;ll create a mangle rule that marks our packets

<pre class="qoate-code">/ip firewall mangle
add action=mark-routing chain=prerouting disabled=no dst-address-list=OVPN \
new-routing-mark=OVPN-mark passthrough=yes src-address=0.0.0.0/0
</pre>

Once that is done , All we need to do is to add a route to route traffic to our VPN link

<pre class="qoate-code">/ip route
add check-gateway=ping disabled=no distance=1 dst-address=0.0.0.0/0 gateway=OVPN-out routing-mark=OVPN-mark scope=10 target-scope=30
</pre>

This should do it , all traffic added to the address list OVPN will be routed over the VPN link &#8211; You can add more IPs/Services if required as well.

Cheers!

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->