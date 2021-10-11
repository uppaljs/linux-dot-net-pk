---
title: Burn a bootable iso to USB Disk on OSX
author: uppal
type: post
date: 2014-12-20T14:38:52+00:00
url: /burn-bootable-iso-usb-disk-osx/
spacious_page_layout:
  - default_layout
categories:
  - DevOps
  - Howtos

---
<pre class="brush: xml; title: ; notranslate" title="">hdiutil convert -format UDRW -o ~/destinationISO.img ~/sourceISO.iso
sudo diskutil umount [/dev/usbdisk path]
sudo dd if=destinationISO.img.dmg of=/dev/disk3 bs=1m
sudo diskutil eject /dev/disk2
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->