---
title: Enable FaceTime on Your Middle-eastern iPhone/iPad
author: uppal
type: post
date: 2014-11-10T18:45:13+00:00
url: /enable-facetime-middle-eastern-iphoneipad/
spacious_page_layout:
  - default_layout
categories:
  - Uncategorized

---
I was stuck with an iPhone purchased in the middle-east where Apple blocks FaceTime &#8211; Turns out that its just a matter of Carrier Bundles &#8211; Apple , from iOS 7 , started signing the carrier bundles , so they couldn&#8217;t be modified by end-customers , any modification would result in change of the signature with the modification over-written on reboot with a standard bundle. A developer team from china enabled modifications by releasing a com-center patch that allows an over-ride for the signature check.

<span style="color: #ff0000;"><strong>You need to have a jail broken phone with Cydia installed.</strong></span>

1. First add the repo for chinasnow in cydia. The repo url is http://apt.chinasnow.net

2. Install com-center-for-ios-8 patch from the repo.

3. Reboot / Respring your phone

4. Open up iFile ( Can be installed from Cydia )

5. Edit **/var/mobile/Library/CarrierBundle.bundle/carrier.plist** , &nbsp;( Use text viewer mode and click the Edit button to edit the file )

6. Add the following on the 5th line in the file,

**<key>AllowsVoIP<&#47;key>  
<true&#47;>  
** 

7. Close the file and reboot your device.

You&#8217;ll have FaceTime available in the options and the FaceTime app icon will pop-up as well. If it doesn&#8217;t work , reboot again.

Cheers!

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->