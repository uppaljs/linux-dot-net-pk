---
title: "Howto Register Your Mobilink Infinity VoIP Number on Softphone"
date: 2008-07-20
categories:
  - Howtos
tags:
  - mobilink
  - sip
  - voip
  - wimax
  - xlite
draft: false
---

Here are the SIP settings if you want to use your VoIP number from Mobilink Infinity on a softphone.

## Disable SIP ALG

You'll have to first logon to your CPE admin interface, browse to **NAT -> ALG** and uncheck **SIP ALG**, then click Apply.

If you use an outdoor unit, do the same with the outdoor CPE as well.

## X-Lite Configuration

Fire up X-Lite and configure it with the following settings:

| Setting | Value |
|---------|-------|
| Server | 10.215.11.250 |
| Port | 5060 |
| UserID | 725XXXX |
| Password | Your WiMAX password |
| Domain | Hauwei.com |
| Authentication User | 725XXXX |

Replace `725XXXX` with your actual Mobilink Infinity VoIP number.

Have fun!
