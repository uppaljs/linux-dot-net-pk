---
title: 'Mbuni – MM7 :: Interconnectivity with Operator'
author: uppal
type: post
date: 2010-08-15T19:38:28+00:00
url: /mbuni-mm7-interconnectivity-with-operator/
categories:
  - Uncategorized

---
Was playing around with Mbuni for a friend of mine , who wanted to use it as a VASP gateway and interconnect it with an operator over MM7. Spent a little while configuring it and thought of posting the configs here for anyone who&#8217;s trying to achieve a similar configuration.

Mbuni consits of

  1. MMS Relay
  2. MMS Proxy
  3. MMS From Email
  4. MMSbox

For this sepecific purpose , only MMSBOX is required. I expect that you&#8217;re already familiar with either <a href="http://kannel.org" target="_blank">Kannel</a> and / or <a href="http://mbuni.org" target="_blank">Mbuni</a> &#8216;s basic configuration.  So I&#8217;ll just focus on MMSBox MM7 integration.

> group = mmsc
> 
> id = local
> 
> mmsc-url = http://username:password@ipaddr:port/mm7/mm7tomms.sh
> 
> incoming-username = incominguser
> 
> incoming-password = incomingpass
> 
> incoming-port = incomingport
> 
> allowed-prefix = Prefixes
> 
> type = soap

The &#8220;mmsc-url&#8221; bit is the most important one as it is used for submit_mms over mm7 , this piece of information is provided by the operator , the extra bit [ /mm7/mm7tomms.sh ] is specific to Jinny&#8217;s MMSC and your submit URL might be different [ pls. confirm this with your operator ].

The incoming user name , password & port are using to receive &#8220;MO-AT&#8221; messages over MM7 from the operator. This information will be provided by you to the operator for configuration on their side . Once a AT MMS is received by Mbuni , it can decide based on the service parameters  on exactly what to do with the mms.

The prefixes are the CC+NDC [ Country Code + National Dialing Codes ] that will be allowed for sending over this mm7 connection. Some samples can be +93;+92;+937;07;+923;03 . Make sure you define it in National [ 07X ] , International [ +937X] & Unknown [ 937X] in order to properly cater for the messages [ both incoming & outgoing  ] .

Fire up mmsbox , hit the url with http://ipaddr:port/cgi-bin/sendmms?username=X&password=Y&to=XXXX&from=YYYY&smil=<INSERT SMIL CONTENTS HERE>

🙂 Cheers!

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->