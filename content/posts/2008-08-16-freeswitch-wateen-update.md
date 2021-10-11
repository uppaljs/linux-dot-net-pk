---
title: 'Freeswitch :: Wateen Update'
author: uppal
type: post
date: 2008-08-16T22:17:07+00:00
url: /freeswitch-wateen-update/
categories:
  - Howtos
tags:
  - freeswitch
  - sip
  - wateen

---
Just a quick update , anthm @ freeswitch.org has fixed the bug in freeswitch that was preventing the switch to register with Wateen&#8217;s gateway. So , go , grab the latest freeswitch code from svn and compile. Here are the settings for your sip_profile

_<gateway name=&#8221;wateen.net&#8221;>  
<param name=&#8221;username&#8221; value=&#8221;021XXXXXX&#8221;/>  
**<param name=&#8221;auth-username&#8221; value=&#8221;021XXXXXX@wateen.net&#8221;/>**  
<param name=&#8221;password&#8221; value=&#8221;&#8221;/>  
<param name=&#8221;register-proxy&#8221; value=&#8221;sip:58.27.240.22:9060&#8243;/>  
<param name=&#8221;register&#8221; value=&#8221;true&#8221;/>  
</gateway>_

🙂 Have fun! and drop a comment if this post was helpful.

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->