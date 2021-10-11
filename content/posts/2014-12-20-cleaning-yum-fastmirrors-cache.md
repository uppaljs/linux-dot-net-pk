---
title: Cleaning up YUM Fastmirrors Cache
author: uppal
type: post
date: 2014-12-20T13:34:06+00:00
url: /cleaning-yum-fastmirrors-cache/
spacious_page_layout:
  - default_layout
categories:
  - DevOps
  - Howtos

---
Quick one for my own reference

<pre class="brush: xml; title: ; notranslate" title="">rm -f /var/cache/yum/timedhosts.txt
yum install &lt;package&gt;
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->