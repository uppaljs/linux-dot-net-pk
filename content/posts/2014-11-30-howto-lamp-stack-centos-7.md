---
title: 'Howto :: LAMP Stack on Centos 7'
author: uppal
type: post
date: 2014-11-30T07:52:15+00:00
url: /howto-lamp-stack-centos-7/
spacious_page_layout:
  - default_layout
categories:
  - DevOps
  - Howtos
tags:
  - apache
  - centos 7
  - howto
  - httpd
  - lamp
  - mysql
  - php

---
Quick and Dirty install below!

Install HTTPD

<pre class="qoate-code">sudo yum install httpd
</pre>

Start and Enable Boot-Startup of httpd

<pre class="qoate-code">sudo systemctl start httpd.service
sudo systemctl enable httpd.service
</pre>

Install & Start / Boot-Start MariaDB ( A drop-in mysql replacement )

<pre class="qoate-code">sudo yum install mariadb-server mariadb
sudo systemctl start mariadb
sudo systemctl enable mariadb.service
</pre>

Secure Your MySQL Installation

<pre class="qoate-code">sudo mysql_secure_installation
</pre>

Add PHP & Mysql Support &#8211; Restart HTTPD

<pre class="qoate-code">sudo yum install php php-mysql
sudo systemctl restart httpd.service
</pre>

Optional Package Install &#8211; You can add PHP Extensions by using the &#8216;yum search&#8217; command to find and &#8216;yum install&#8217; to install.

<pre class="qoate-code">yum search php-
sudo yum install pkg1 pkg2 etc.
</pre>

Finally , Open up ports on FirewallD

<pre class="qoate-code">sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
</pre>

Optional &#8211; mcrypt and imagemagick support

<pre class="qoate-code">yum install ImageMagick ImageMagick-devel
yum install php-mcrypt*
</pre>

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->