---
title: Cisco ASA VPN Woes!
author: uppal
type: post
date: 2010-08-17T21:19:00+00:00
url: /cisco-asa-vpn-woes/
categories:
  - Howtos
  - Rants

---
This is to document the config for outside access to VPN , incase i forget again!

> access-list outside_nat extended permit ip [vpn client network] 255.255.255.0 any
> 
> global (outside) 1 interface
> 
> nat (outside) 1 access-list outside_nat
> 
> group-policy DfltGrpPolicy attributes
> 
> dns-server value [dns server]
> 
> nem enable
> 
> group-policy VPN attributes
> 
> split-tunnel-policy tunnelall
> 
> split-tunnel-network-list none

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->