---
title: "Mbuni - MM7: Interconnectivity with Operator"
date: 2010-08-15
categories:
  - Howtos
tags:
  - mbuni
  - mm7
  - mms
  - telecom
draft: false
---

Was playing around with Mbuni for a friend of mine, who wanted to use it as a VASP gateway and interconnect it with an operator over MM7. Spent a little while configuring it and thought of posting the configs here for anyone who's trying to achieve a similar configuration.

Mbuni consists of:

1. MMS Relay
2. MMS Proxy
3. MMS From Email
4. MMSbox

For this specific purpose, only MMSBOX is required. I expect that you're already familiar with either Kannel and/or Mbuni's basic configuration. So I'll just focus on MMSBox MM7 integration.

```
group = mmsc
id = local
mmsc-url = http://username:password@ipaddr:port/mm7/mm7tomms.sh
incoming-username = incominguser
incoming-password = incomingpass
incoming-port = incomingport
allowed-prefix = Prefixes
type = soap
```

The `mmsc-url` bit is the most important one as it is used for submit_mms over MM7. This piece of information is provided by the operator. The extra bit `/mm7/mm7tomms.sh` is specific to Jinny's MMSC and your submit URL might be different (please confirm this with your operator).

The incoming username, password & port are used to receive "MO-AT" messages over MM7 from the operator. This information will be provided by you to the operator for configuration on their side. Once an AT MMS is received by Mbuni, it can decide based on the service parameters on exactly what to do with the MMS.

The prefixes are the CC+NDC (Country Code + National Dialing Codes) that will be allowed for sending over this MM7 connection. Some samples can be `+93;+92;+937;07;+923;03`. Make sure you define it in National (07X), International (+937X) & Unknown (937X) in order to properly cater for the messages (both incoming & outgoing).

Fire up mmsbox, hit the URL with:

```
http://ipaddr:port/cgi-bin/sendmms?username=X&password=Y&to=XXXX&from=YYYY&smil=
```

Cheers!
