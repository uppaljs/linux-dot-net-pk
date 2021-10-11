---
title: Remove /home partition on CentOS 6
author: uppal
type: post
date: 2015-05-01T08:50:18+00:00
url: /remove-home-partition-on-centos-6/
spacious_page_layout:
  - default_layout
categories:
  - DevOps
  - Howtos

---
Centos by default creates a separate LVM with /home , which is un-necessary in certain installs , to remove it, follow the procedure

To check the current status:

<pre class="brush: bash; title: ; notranslate" title="">[root@localhost ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root
                       50G  1.2G   46G   3% /
tmpfs                 935M     0  935M   0% /dev/shm
/dev/xvda1            485M   55M  405M  12% /boot
/dev/mapper/VolGroup-lv_home
                       45G  180M   43G   1% /home
</pre>

Executing the changes:

<pre class="brush: bash; title: ; notranslate" title="">umount /home
lvm lvremove /dev/mapper/VolGroup-lv_home
lvm lvresize -l+100%FREE /dev/mapper/VolGroup-lv_root
resize2fs /dev/mapper/VolGroup-lv_root
</pre>

Checking size again:

<pre class="brush: bash; title: ; notranslate" title="">[root@localhost /]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root
                       95G  1.2G   89G   2% /
tmpfs                 935M     0  935M   0% /dev/shm
/dev/xvda1            485M   55M  405M  12% /boot
</pre>

Modifying /etc/fstab to remove the mount point for /home

<pre class="brush: bash; title: ; notranslate" title="">#
# /etc/fstab
# Created by anaconda on Thu Jul  3 11:57:44 2014
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup-lv_root /                       ext4    defaults        1 1
UUID=bba0a5bb-7a8a-4d63-896f-c38b23422de3 /boot                   ext4    defaults        1 2
/dev/mapper/VolGroup-lv_home /home                   ext4    defaults        1 2
/dev/mapper/VolGroup-lv_swap swap                    swap    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->