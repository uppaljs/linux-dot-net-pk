---
title: 'Mobilink Infinity :: Freeswitch :: One Way Audio'
author: uppal
type: post
date: 2008-07-22T10:53:15+00:00
url: /mobilink-infinity-freeswitch-one-way-audio/
categories:
  - Howtos
tags:
  - freeswitch
  - infinity
  - issue
  - mobilink
  - mobilink voip
  - oneway audio
  - voip

---
If you are facing one way audio issue with freeswitch for all outgoing calls via mobilink infinity trunk , all you need to do is to edit **conf/vars.xml** and set **EXT\_SIP\_IP** & **EXT\_RTP\_IP** to 192.168.100.1 . That&#8217;s it , your one way audio issue will be resolved 🙂 .

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->