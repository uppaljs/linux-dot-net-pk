---
title: Asterisk Config for PTCL SmartLink App
author: uppal
type: post
date: 2014-11-27T13:09:30+00:00
url: /asterisk-config-ptcl-smartlink-app/
spacious_page_layout:
  - default_layout
categories:
  - Howtos
  - voip
tags:
  - asterisk
  - configuration
  - howto
  - pjsip
  - ptcl
  - sip
  - voip

---
PTCL has recently launched an app called &#8220;SmartLink&#8221; , Which provides you access to your phone line over SIP , essentially providing landline access from anywhere.

Since the land-line I have is only used for DSL and SmartTV, I wanted to set it up on SIP , so I can access it from any device.

PTCL uses Huawei SoftX 3000 Soft Switches for this service , I have had experience on these switches from my previous jobs and I know that it requires PRACK support to start off , In addition , while configuring , I noticed that the user_agent also needs to be set to a certain value for the registration to be successful.

I used chan_pjsip and asterisk 13 for this setup. Below is the configuration that works!

<pre class="qoate-code">[ptcl]

type=registration 
transport=simpletrans 
outboundauth=ptcl 
serveruri=sip:SIPSERVER 
clienturi=sip:USERNAME@SIPSERVER 
expiration=900 
contactuser=USERNAME 
supportpath=yes

[ptcl]

type=auth 
auth_type=userpass 
password=PASSWORD 
username=USERNAME

[ptcl]

type=aor 
contact=sip:SIPSERVER:5060

[ptcl]

type=endpoint 
transport=simpletrans 
context=from-ptcl 
disallow=all 
allow=ulaw 
outboundauth=ptcl 
fromdomain = SIPSERVER 
fromuser = USERNAME 
dtmfmode = rfc4733 
force_rport = yes 
aors=ptcl 
timers_sess_expires=900 
100rel=yes

[ptcl] type=identify 
endpoint=ptcl 
match=SIPSERVER
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->