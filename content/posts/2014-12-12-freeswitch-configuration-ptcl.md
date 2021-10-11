---
title: Configure Your PTCL SmartLink SIP Account On Freeswitch
author: uppal
type: post
date: 2014-12-12T12:12:32+00:00
url: /freeswitch-configuration-ptcl/
spacious_page_layout:
  - default_layout
categories:
  - Howtos
  - voip
tags:
  - asterisk
  - freeswitch
  - ptcl
  - sip
  - smartlink
  - voip

---
Following up on my previous post on how to configure your PTCL Smart Link SIP account on asterisk, below is the configuration I used for PTCL &#8211; Just replace USERNAME and PASSWORD with your credentials.

<pre class="brush: xml; title: ; notranslate" title="">&lt;gateway name="ptcl"&gt;
		&lt;param name="username" value="USERNAME" /&gt;
		&lt;param name="realm" value="59.103.224.34" /&gt;
		&lt;param name="from-user" value="USERNAME" /&gt;
		&lt;param name="from-domain" value="59.103.224.34" /&gt;
		&lt;param name="password" value="PASSWORD" /&gt;
		&lt;param name="extension" value="USERNAME" /&gt;
		&lt;param name="expire-seconds" value="900" /&gt;
		&lt;param name="register" value="true" /&gt;
		&lt;param name="register-transport" value="udp" /&gt;
		&lt;param name="retry-seconds" value="30"/&gt;
		&lt;param name="extension-in-contact" value="true" /&gt;
		&lt;param name="caller-id-in-from" value="false" /&gt;
		&lt;param name="enable-100rel" value="true"/&gt;
&lt;/gateway&gt;
</pre>

You will also need to change your user agent , This can be done by adding the below parameter in sip_profiles/external.xml

<pre class="brush: xml; title: ; notranslate" title="">&lt;param name="user-agent-string" value="CSipSimple_hwmt1-u06-16/r2"/&gt;
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->