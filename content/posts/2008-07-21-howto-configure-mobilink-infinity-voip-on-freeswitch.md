---
title: Howto configure Mobilink Infinity VoIP on Freeswitch
author: uppal
type: post
date: 2008-07-21T20:23:47+00:00
url: /howto-configure-mobilink-infinity-voip-on-freeswitch/
categories:
  - Howtos
tags:
  - asterisk
  - freeswitch
  - huawei
  - mobilink
  - softx3000
  - voip
  - wimax

---
You can configure your mobilink infinity VoIP number on [Freeswitch][1] aswell , Freeswitch is a very nice alternative to Asterisk with a SIP stack that actually **works! BKW @ #freeswitch on irc.freenode.net** helped me with this 🙂

Here is the **conf/sip_profiles/external/mobilink.xml** settings

_<include>  
<gateway name=&#8221;mobimax&#8221;>  
<param name=&#8221;username&#8221; value=&#8221;725xxxx&#8221;/>  
<param name=&#8221;password&#8221; value=&#8221;Your_Password&#8221;/>  
<param name=&#8221;proxy&#8221; value=&#8221;10.215.11.250&#8243;/>  
</gateway>  
</include></p> 

</em>

Here&#8217;s the dialplan entry **conf/dialplan/extensions/000_myext.xml**

_<include>  
<extension name=&#8221;enum&#8221;>  
<condition field=&#8221;destination_number&#8221; expression=&#8221;^(0300d+)$&#8221;>  
<action application=&#8221;bridge&#8221; data=&#8221;sofia/gateway/mobimax/$1&#8243;/>  
</condition>  
</extension>  
</include>_

Oh by the way , for this, make sure the SIP ALG on your outdoor cpe is turned on , while the indoor one needs to be off 🙂

Have fun!

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->

 [1]: http://freeswitch.org