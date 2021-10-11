---
title: 'Poor Man’s GSM BTS :: Nuand’s BladeRF & OpenBTS 5 Setup Instructions'
author: uppal
type: post
date: 2015-01-16T15:05:54+00:00
url: /poor-mans-gsm-bts-nuands-bladerf-openbts-5-setup-instructions/
spacious_page_layout:
  - default_layout
categories:
  - Uncategorized

---
!!!! THIS IS FOR EDUCATIONAL PURPOSES ONLY !!!!

These instructions are for setting up and running OpenBTS with Nuand&#8217;s BladeRF x40 Board , These are a work in progress and should be taken as is. This was possible with the help of @mambrus on #bladerf and @rwr on #bladerf.[1] and is a collection of different how-tos linked at the end of the post.

OS Used : Ubuntu 12.04 LTS Server 32 bit , The ISO can be downloaded from [here][1]

1. Download & Install Ubuntu 12.04 32 Bit , Do a minimalistic install with OpenSSH server.  
2. Update git

<pre class="brush: bash; title: ; notranslate" title="">$ sudo apt-get install software-properties-common python-software-properties
$ sudo add-apt-repository ppa:git-core/ppa (press enter to continue)
$ sudo apt-get update
$ sudo apt-get install git
</pre>

**Pre-Reqs Installation**

Once this is setup , its time to setup the pre-requisites 

Copy and Paste this , and run it &#8211; it will install the dependencies:

<pre class="brush: bash; title: ; notranslate" title="">sudo apt-get install $(
    wget -qO - https://raw.githubusercontent.com/RangeNetworks/dev/master/build.sh | \
    grep installIfMissing | \
    grep -v "{" | \
    cut -f2 -d" ")
</pre>

Another way to install the dependencies is to look for IFMissing in the above file and install them via apt-get or you can build from source.

Two of the packages would fail (libzmq3 & libzmq3-devel) &#8211; They can be installed on Ubuntu with the following commands

<pre class="brush: bash; title: ; notranslate" title="">$ sudo add-apt-repository ppa:chris-lea/zeromq
$ sudo apt-get update
$ sudo apt-get install libzmq3-dbg libzmq3-dev 
</pre>

The next step is to build **uhd** , that be done by compiling GNURadio from source , running the command below will do that , and is going to a take a **LOT** of time , depending on your computer. [ http://code.ettus.com/redmine/ettus/projects/uhd/wiki/UHD_Linux <= You can following this link as well for UHD] [bash] $ wget http://www.sbrac.org/files/build-gnuradio && chmod a+x ./build-gnuradio && ./build-gnuradio [/bash] Once that is done , you'll get either a success or a failure message , you can use the -v option to do a verbose install again to find out and fix the issue. Next step is to install some more dependencies for OpenBTS , they are the following 1. libgsm1-dev 2. asterisk-dev 3. asterisk-config On ubuntu , this can be done by [bash] $ sudo apt-get install libgsm1-dev asterisk-dev asterisk-config [/bash] You may or maynot need to update **libusb** as well , depending on your system , head over to www.libusb.org and download the latest copy in /usr/src and compile. copy it over to /usr/lib/x86_64-linux-gnu/libusb.so after taking a backup of the original file.

**OpenBTS Installation**

Once you have all the dependencies sorted out , Its time to install openBTS

1. Create a directory for OpenBTS (obts in our case )  
2. Change to that directory and run the following

<pre class="brush: bash; title: ; notranslate" title="">#!/bin/bash

git clone https://github.com/RangeNetworks/openbts.git
git clone https://github.com/RangeNetworks/smqueue.git
git clone https://github.com/RangeNetworks/subscriberRegistry.git

#From here and downwards you can copy&paste (that's why the ';' are for)
for D in *; do (
    echo $D;
    echo "=======";
    cd $D;
    git clone https://github.com/RangeNetworks/CommonLibs.git;
    git clone https://github.com/RangeNetworks/NodeManager.git);
done;
git clone https://github.com/RangeNetworks/libcoredumper.git;
git clone https://github.com/RangeNetworks/liba53.git
</pre>

3. Build libcoredumper

<pre class="brush: bash; title: ; notranslate" title="">cd libcoredumper;
./build.sh && \
   sudo dpkg -i *.deb;
cd ..
</pre>

4. Build liba53

<pre class="brush: bash; title: ; notranslate" title="">cd liba53;
make && \
   sudo make install;
cd ..;
</pre>

5. In the same directory , check out YateBTS

<pre class="brush: bash; title: ; notranslate" title="">svn checkout http://voip.null.ro/svn/yatebts/trunk yatebts 
</pre>

6. The next step is to remove autoloading of the FPGA since we will be loading it permanently. To do that , open up 

<pre class="brush: bash; title: ; notranslate" title="">vim ./yatebts/mbts/TransceiverRAD1/bladeRFDevice.cpp
</pre>

and add  **#ifdef NEVER on line 108 and #endif on line 129** &#8211; Since both lines are empty , future patches should work as well.

7. Change to YateBTS directory and run autogen.sh

<pre class="brush: bash; title: ; notranslate" title="">$ cd opbts/yatebts
$ ./autogen.sh
</pre>

8. This will generate a configure script , Now if you run the configure script immediately , it will fail while looking for YATE , so open up the configure file

<pre class="brush: bash; title: ; notranslate" title="">$vim configure
</pre>

Look for  **as\_fn\_err $? &#8220;Could not find Yate&#8221; &#8220;$LINENO&#8221; 5** and change it to **as\_fn\_warn $? &#8220;Could not find Yate&#8221; &#8220;$LINENO&#8221; 5**

9. Rerun configure 

<pre class="brush: bash; title: ; notranslate" title="">$ ./configure
</pre>

10. You only need two directories from YateBTS 

a) Peering

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/yatebts/mbts/Peering
$ make
</pre>

b) TransceiverRAD1

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/yatebts/mbts/TransceiverRAD1
$ make
</pre>

11. Copy the two binaries over to OpenBTS folder.

<pre class="brush: bash; title: ; notranslate" title="">$ cd ..
$ cp ./yatebts/mbts/TransceiverRAD1/transceiver-bladerf openbts/apps/
$ cd openbts/apps/
$ ln -sf transceiver-bladerf transceiver
</pre>

12. Build OpenBTS

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/openbts
$ ./autogen.sh
$ ./configure --with-uhd
$ make
</pre>

13. Next step is to configure the SQL-lite Database , For bladeRF , a few modifications are required. They can be done as

<pre class="brush: bash; title: ; notranslate" title="">$ vim /home/openbts/obts/openbts/apps/OpenBTS.example.sql
</pre>

Look for the following and replace their values

<pre class="brush: xml; title: ; notranslate" title="">GSM.Radio.RxGain from 47 to 5 
GSM.Radio.PowerManager.MaxAttenDB to 35
GSM.Radio.PowerManager.MinAttenDB to 35
</pre>

Once that is done , proceed to the next step

14. Create OpenBTS configuration directory

<pre class="brush: bash; title: ; notranslate" title="">$ sudo mkdir /etc/OpenBTS
</pre>

15. Install DB , from OpenBTS source directory

<pre class="brush: bash; title: ; notranslate" title="">$ sudo sqlite3 -init ./apps/OpenBTS.example.sql /etc/OpenBTS/OpenBTS.db ".quit"
</pre>

16. Once that is done , you can test it by running the following command

<pre class="brush: bash; title: ; notranslate" title="">$ sqlite3 /etc/OpenBTS/OpenBTS.db .dump
</pre>

If you see a lot of output &#8211; Then the installation is successful , Proceed to the next step

17. Run OpenBTS by using the following commands

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/openbts/apps
$ sudo ./OpenBTS
</pre>

If you see **system ready** &#8211; Your BTS is up and running and now if you do a search for networks from your phone , it should show Test PLMN or 00101 in your phone.  
[<img src="http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_20_34_Pro1-169x300.jpg" alt="WP_20150116_18_20_34_Pro" width="169" height="300" class="alignnone size-medium wp-image-232" srcset="http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_20_34_Pro1-169x300.jpg 169w, http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_20_34_Pro1-576x1024.jpg 576w, http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_20_34_Pro1.jpg 918w" sizes="(max-width: 169px) 100vw, 169px" />][2]

[<img src="http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_19_51_Pro1-300x169.jpg" alt="WP_20150116_18_19_51_Pro" width="300" height="169" class="alignnone size-medium wp-image-233" srcset="http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_19_51_Pro1-300x169.jpg 300w, http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_19_51_Pro1-1024x576.jpg 1024w" sizes="(max-width: 300px) 100vw, 300px" />][3]  
18. If you see the above , shutdown openBTS and proceed to the next step for installation of subscriber registry, sipauthserve and smqueue , both of these are required for running openBTS &#8211; Without them , the phone will not register to the test network.

19. To setup the Subscriber Registry database you must first create the file path the db will reside in. By default, this is /var/lib/asterisk/sqlite3dir . Create that directory

<pre class="brush: bash; title: ; notranslate" title="">$ sudo mkdir -p /var/lib/asterisk/sqlite3dir
</pre>

20. To build sipauthserve

<pre class="brush: bash; title: ; notranslate" title="">$ cd subscriberRegistry
$ ./autogen.sh
$ ./configure
$ make
</pre>

This will create a binary sipauthserve in **/home/openbts/obts/subscriberRegistry/apps**

21. The next step is to configure sipauthserve , this can be done by

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/subscriberRegistry
$ sudo sqlite3 -init subscriberRegistry.example.sql /etc/OpenBTS/sipauthserve.db ".quit"
</pre>

22. The next step is to build and install SMQUEUE , SMQUEUE will complain about missing SubscriberRegistry.h &#8211; Which can be fixed by creating a symlink to subscriberRegistry directory in smqueue build path.

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/smqueue
$ ln -s /home/openbts/obts/subscriberRegistry/ SR
$ autoreconf -i
$ ./configure
$ make
</pre>

23. Once this is done , SMQUEUE configuration needs to be imported.

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/smqueue
$ sudo sqlite3 -init smqueue/smqueue.example.sql /etc/OpenBTS/smqueue.db ".quit"
</pre>

 **bladeRF Firmware Update & FPGA Loading** 

24. Head over to [https://github.com/Nuand/bladeRF/wiki/Upgrading-bladeRF-firmware] and update the firmware.

25. Once that is done , Head over to [http://www.nuand.com/fpga.php] to download the appropriate FPGA image for your board.

26. Load the FPGA image 

<pre class="brush: bash; title: ; notranslate" title="">$ bladeRF-cli -L &lt;path to fpga image file&gt;
</pre>

 **patience , lots of it, or you may damage your board!** 

27. Once all of this is done , start the services one by one

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/smqueue
$ sudo ./smqueue &
</pre>

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/subscriberRegistry/apps
$ sudo ./sipauthserve &
</pre>

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/openbts/apps
$ sudo ./OpenBTS &
</pre>

25. Start OpenBTSCLI by running 

<pre class="brush: bash; title: ; notranslate" title="">$ cd /home/openbts/obts/openbts/apps/
$ sudo ./OpenBTSCLI
</pre>

28. By default , OpenBTS will not accept your registration , till you do the following

a) Enter a regex for your IMSI  
b) Allow all IMSIs to be accepted by setting

<pre class="brush: bash; title: ; notranslate" title="">config Control.LUR.OpenRegistration .*
</pre>

in the openBTS cli.

**THIS WILL ALLOW ALL OF THE HANDSETS REGISTERING TO YOUR BTS &#8211; INCLUDING YOUR NEIGHBOURS! DONOT USE IT PERMANENTLY !!!!**

29. You should now be able to search your network on your phone , and latch onto it &#8211; Dial 600 to do an echo test on Asterisk from your phone!

30. Please ensure that you follow all local laws!

[1] https://github.com/Nuand/bladeRF/wiki/Minimalistic-build-and-run-test-for-OpenBTS-5  
[2] https://wush.net/trac/rangepublic/wiki/BuildInstallRun#ConfiguringOpenBTS  
[3] https://wush.net/trac/rangepublic/wiki/CommonErrors  
[4] http://openbts.org/w/index.php/Main_Page  
[5] https://github.com/Nuand/bladeRF/wiki/Upgrading-bladeRF-firmware

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->

 [1]: http://sunsite.rediris.es/mirror/ubuntu-releases/precise/ubuntu-12.04.4-server-i386.iso
 [2]: http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_20_34_Pro1.jpg
 [3]: http://linux.net.pk/blog/wp-content/uploads/2015/01/WP_20150116_18_19_51_Pro1.jpg