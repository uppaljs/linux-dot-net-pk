---
title: 'Freeswitch :: Register-proxy Bug'
author: uppal
type: post
date: 2008-07-27T06:55:08+00:00
url: /freeswitch-register-proxy-bug/
categories:
  - freeswitch
tags:
  - bug
  - freeswitch
  - register-proxy
  - wateen

---
While trying to configure wateen sip on my freeswitch install , i encountered a bug , aparently wateen has a broken implementation , they dont have srv records for their domain wateen.net, but their sip server / ims envoirnment seems to be forcing it , in the sip &#8211; uri. Here&#8217;s exactly whats broken

**Working &#8211; Eyebeam**

REGISTER sip:wateen.net SIP/2.0  
To: Wateen User<sip:[021xxxxx@wateen.net][1]>  
From: Wateen User<sip:[021xxxxx@wateen.net][1]>;tag=90298206  
Via: SIP/2.0/UDP 192.168.1.51:6473;branch=z9hG4bK-d87543-704653288-1&#8211;d87543-;rport  
Call-ID: 3736784a1d468701  
CSeq: 1 REGISTER  
Contact: <sip:[021xxxxx@192.168.1.51][2]:6473>  
Expires: 3600  
Max-Forwards: 70  
Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, NOTIFY, MESSAGE, SUBSCRIBE, INFO  
User-Agent: eyeBeam release 3007n stamp 17816  
Content-Length: 0 ****

**Not Working &#8211; Freeswitch**

REGISTER sip:58.27.240.22:9060;transport=udp SIP/2.0  
Via: SIP/2.0/UDP 67.18.xx.xxx:5080;rport;branch=z9hG4bKB0majXrF4mBem  
Max-Forwards: 70  
From: <sip:[021xxxx@wateen.net][1];transport=udp>;tag=BQZeg23XQQX5r  
To: <sip:[021xxxxx@wateen.net][1];transport=udp>  
Call-ID: 6d64ca19-d640-122b-1685-e6fdbcfd8af8  
CSeq: 102450077 REGISTER  
Contact: <sip:[021xxxxx@67.18.89.4][3]:5080;transport=udp>  
Expires: 3600  
User-Agent: FreeSWITCH-mod_sofia/1.0.1-exported  
Allow: INVITE, ACK, BYE, CANCEL, OPTIONS, PRACK, MESSAGE, SUBSCRIBE, NOTIFY, REFER, UPDATE, REGISTER, INFO  
Supported: 100rel, timer, precondition, path, replaces  
Content-Length: 0

so upon more investigation , found out that register-proxy is putting up the ip of the server in the request uri instead of using NTATAG_PROXY .

I have openned up a bug report with freeswitch , lets hope it gets resolved quickly 😉

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->

 [1]: mailto:0218000342@wateen.net
 [2]: mailto:0218000342@192.168.1.51
 [3]: mailto:0218000342@67.18.89.4