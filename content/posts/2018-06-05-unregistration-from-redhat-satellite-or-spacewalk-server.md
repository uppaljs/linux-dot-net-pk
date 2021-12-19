---
title: Unregistration from Redhat Satellite or Spacewalk Server
author: uppal
type: post
date: 2018-06-05T12:56:52+00:00
url: /unregistration-from-redhat-satellite-or-spacewalk-server/
spacious_page_layout:
  - default_layout
categories:
  - Howtos

---
Ideally use
```console
# subscription-manager unregister
```

Otherwise:
```console
# rm /etc/sysconfig/rhn/systemid
```
should do the trick.
