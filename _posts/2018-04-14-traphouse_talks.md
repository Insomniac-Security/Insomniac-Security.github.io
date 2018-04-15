---
title: Red and Blue Teaming a Modern House
published: true
layout: post
description: Talks on protecting your home or breaking into one.
author: David E. Switzer
---
## Intro

A couple years ago, I did a talk at BSides:Orlando called ["Breadcrumbs"](https://www.youtube.com/watch?v=HzQHWUM8cNo).   It was about collecting wifi probes, semi-auto sorting out which is which (who, more accurately), and alerting you when a few people showed up.   This was entirely born out of having a boss that worked out of Canada regularly showing up to the office -- surprise!  Jonathan and I needed protection!

Later, Jonathan and I combined my fascination w/ RF metadata w/ his fledgling love of home-automation, and did a talk called ["The Trap House: Making Your Home As Paranoid As Your Are"](https://www.youtube.com/watch?v=OWUu0gQTKkM).   This was a "stable" talk at Derbycon 2017.

At BSides:Tampa 2018, we presented a talk called ["Modern Day Vandals and Thieves: RF Edition"](https://www.youtube.com/watch?v=O01ppGIWjr0), which laid out ways to identify valuables inside of a home, a few ways of detecting if people were home, and detecting if alarm systems were present.

At this point, I realized that we were basically red/blue teaming homes, and we embraced it.   Thus Bsides:Orlando 2018's talk "Redteaming the Traphouse".  This combines some aspects of all the talks, adds some extra ideas, some new stuff about an alarm system that pulled us down a rabbit hole, and some other random toys.

More to come on this topic -- including a future blog going over some of the items in these talks.  Let us know if you have any questions!


## The Promised Slides/Files [tm]

Slides: ["Redteaming The Traphouse"](https://github.com/Insomniac-Security/Insomniac-Security.github.io/blob/master/static/docs/BsidesOrl2018_Redteaming-the-traphouse.pdf)  
["house_hide.py"](https://github.com/violentlydave/GatosGuardianes/blob/master/house_hide.py) - the program to spew out probes from fake IoT devices.  
["callsomeonesaysomething.sh"](https://github.com/violentlydave/GatosGuardianes/blob/master/callsomeonesaysomething.sh) - a script to automate sending synthesized voice calls for alerts/etc.
[ZWave captures from Jonathan's Wink System](https://www.youtube.com/watch?v=GL-8XuoxuaQ) - ZWave captures used for replay attacks mentioned in the talk.  

## Projects referenced in the slides:

Some of the fun projects and software we discussed:

[Home Assistant](https://www.home-assistant.io/) - A vendor agnostic middleman for your home-automation needs!  
[RTL-SDR.com](http://www.rtl-sdr.com) - A fantastic SDR site.  I guess I didn't need to hotlink that, really.  
[BLEah](https://github.com/evilsocket/bleah) - BTLE sniffer/prober/all around party good time.  
[Killerbee](https://github.com/riverloopsec/killerbee) - Tools for attacking Zigbee.  
[Killerzee](https://github.com/riverloopsec/killerzee) - Tools for attacking Z-Wave.  
[RTL-AMR](https://github.com/bemasher/rtlamr) - RTL-SDR receiver for decoding AMR power meters.  
[GQRX](http://gqrx.dk) - A GnuRADIO based graphical radio receiver.  
[URH](https://github.com/jopohl/urh) - Universal Radio Hacker - for capturing, analzying and replaying RF signals.  

