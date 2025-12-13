---
title: "Poor Man's GSM BTS :: Nuand's BladeRF & OpenBTS 5 Setup Instructions"
date: 2015-01-16
categories:
  - Howtos
  - SDR
tags:
  - gsm
  - openbts
  - bladerf
  - sdr
  - telecom
draft: false
---

**!!!! THIS IS FOR EDUCATIONAL PURPOSES ONLY !!!!**

These instructions are for setting up and running OpenBTS with Nuand's BladeRF x40 Board. These are a work in progress and should be taken as is. This was possible with the help of @mambrus on #bladerf and @rwr on #bladerf, and is a collection of different how-tos linked at the end of the post.

**OS Used:** Ubuntu 12.04 LTS Server 32 bit, The ISO can be downloaded from [here](http://sunsite.rediris.es/mirror/ubuntu-releases/precise/ubuntu-12.04.4-server-i386.iso)

1. Download & Install Ubuntu 12.04 32 Bit, Do a minimalistic install with OpenSSH server.
2. Update git

## Pre-Reqs Installation

Once this is setup, its time to setup the pre-requisites.

Copy and paste the dependencies script and run it – it will install the dependencies. Another way to install the dependencies is to look for `IFMissing` in the file and install them via apt-get or you can build from source.

Two of the packages would fail (`libzmq3` & `libzmq3-devel`) – They can be installed on Ubuntu with the following commands.

The next step is to build **uhd**, that can be done by compiling GNURadio from source. Running the command below will do that, and is going to take a **LOT** of time, depending on your computer.

You can also follow the [UHD Linux guide](http://code.ettus.com/redmine/ettus/projects/uhd/wiki/UHD_Linux).

```bash
wget http://www.sbrac.org/files/build-gnuradio && chmod a+x ./build-gnuradio && ./build-gnuradio
```

Once that is done, you'll get either a success or a failure message. You can use the `-v` option to do a verbose install again to find out and fix the issue.

Next step is to install some more dependencies for OpenBTS:

1. libgsm1-dev
2. asterisk-dev
3. asterisk-config

On Ubuntu, this can be done by:

```bash
sudo apt-get install libgsm1-dev asterisk-dev asterisk-config
```

You may or may not need to update **libusb** as well, depending on your system. Head over to [www.libusb.org](http://www.libusb.org) and download the latest copy in `/usr/src` and compile. Copy it over to `/usr/lib/x86_64-linux-gnu/libusb.so` after taking a backup of the original file.

## OpenBTS Installation

Once you have all the dependencies sorted out, it's time to install OpenBTS.

1. Create a directory for OpenBTS (`obts` in our case)
2. Change to that directory and run the following

3. Build libcoredumper

4. Build liba53

5. In the same directory, check out YateBTS

6. The next step is to remove autoloading of the FPGA since we will be loading it permanently. To do that, open up the relevant file and add `#ifdef NEVER` on line 108 and `#endif` on line 129 – Since both lines are empty, future patches should work as well.

7. Change to YateBTS directory and run `autogen.sh`

8. This will generate a configure script. Now if you run the configure script immediately, it will fail while looking for YATE, so open up the configure file.

Look for:
```
as_fn_err $? "Could not find Yate" "$LINENO" 5
```

And change it to:
```
as_fn_warn $? "Could not find Yate" "$LINENO" 5
```

9. Rerun configure

10. You only need two directories from YateBTS:
    - Peering
    - TransceiverRAD1

11. Copy the two binaries over to OpenBTS folder.

12. Build OpenBTS

13. Next step is to configure the SQLite Database. For bladeRF, a few modifications are required. Look for the following and replace their values.

Once that is done, proceed to the next step.

14. Create OpenBTS configuration directory

15. Install DB from OpenBTS source directory

16. Once that is done, you can test it by running the following command.

If you see a lot of output – Then the installation is successful. Proceed to the next step.

17. Run OpenBTS by using the following commands.

If you see **system ready** – Your BTS is up and running and now if you do a search for networks from your phone, it should show Test PLMN or 00101 in your phone.

18. If you see the above, shutdown OpenBTS and proceed to the next step for installation of subscriber registry, sipauthserve and smqueue. Both of these are required for running OpenBTS – Without them, the phone will not register to the test network.

19. To setup the Subscriber Registry database you must first create the file path the db will reside in. By default, this is `/var/lib/asterisk/sqlite3dir`. Create that directory.

20. To build sipauthserve.

This will create a binary sipauthserve in `/home/openbts/obts/subscriberRegistry/apps`

21. The next step is to configure sipauthserve.

22. The next step is to build and install SMQUEUE. SMQUEUE will complain about missing `SubscriberRegistry.h` – Which can be fixed by creating a symlink to subscriberRegistry directory in smqueue build path.

23. Once this is done, SMQUEUE configuration needs to be imported.

## bladeRF Firmware Update & FPGA Loading

24. Head over to the [bladeRF firmware upgrade wiki](https://github.com/Nuand/bladeRF/wiki/Upgrading-bladeRF-firmware) and update the firmware.

25. Once that is done, head over to [http://www.nuand.com/fpga.php](http://www.nuand.com/fpga.php) to download the appropriate FPGA image for your board.

26. Load the FPGA image.

**Patience, lots of it, or you may damage your board!**

27. Once all of this is done, start the services one by one.

28. Start OpenBTSCLI by running.

29. By default, OpenBTS will not accept your registration, till you do the following:

a) Enter a regex for your IMSI

b) Allow all IMSIs to be accepted by setting the appropriate configuration in the OpenBTS CLI.

**THIS WILL ALLOW ALL OF THE HANDSETS REGISTERING TO YOUR BTS – INCLUDING YOUR NEIGHBOURS! DO NOT USE IT PERMANENTLY!!!!**

30. You should now be able to search your network on your phone, and latch onto it – Dial 600 to do an echo test on Asterisk from your phone!

31. **Please ensure that you follow all local laws!**

## References

1. [Minimalistic build and run test for OpenBTS 5](https://github.com/Nuand/bladeRF/wiki/Minimalistic-build-and-run-test-for-OpenBTS-5)
2. [BuildInstallRun - Configuring OpenBTS](https://wush.net/trac/rangepublic/wiki/BuildInstallRun#ConfiguringOpenBTS)
3. [Common Errors](https://wush.net/trac/rangepublic/wiki/CommonErrors)
4. [OpenBTS Main Page](http://openbts.org/w/index.php/Main_Page)
5. [Upgrading bladeRF firmware](https://github.com/Nuand/bladeRF/wiki/Upgrading-bladeRF-firmware)
