---
layout: page
title: "JUNOS (Juniper) Flaw Exposes Core Routers to Kernel Crash"
categories: 
  - security
  - networking
tags: 
  - juniper
  - tcp
  - option-101
  - networking 
  - crash
  - junos
author: jrossi
teaser: |
    A report has been received from Juniper at 4:25pm under bulletin PSN-2010-01-623 that a 
    crafted malformed TCP field option in the TCP header of a packet will cause the JUNOS kernel 
    to core (crash). In other words the kernel on the network device (gateway router) will 
    crash and reboot if a packet containing this crafted option is received on a listening 
    TCP port. The JUNOS firewall filter is unable to filter a TCP packet with this issue. 
    Juniper claims this issue as exploit was identified during investigation of a vendor 
    interoperability issue.
---


A report has been received from Juniper at 4:25pm under bulletin
PSN-2010-01-623 that a crafted malformed TCP field option in the TCP
header of a packet will cause the JUNOS kernel to core (crash). In other
words the kernel on the network device (gateway router) will crash
and reboot if a packet containing this crafted option is received on
a listening TCP port. The JUNOS firewall filter is unable to filter a
TCP packet with this issue. Juniper claims this issue as exploit was
identified during investigation of a vendor interoperability issue.

There is talk that backbone Internet providers have been quickly        
patching this issue since yesterday night.                              

### TCP Header Option Space

> "Options occupy space at the end of the TCP header. All options are included in the 
> checksum. An option may begin on any byte boundary. The TCP header must be padded with 
> zeros to make the header length a multiple of 32 bits." 
>
> (Source: http://www.networksorcery.com/enp/protocol/tcp.htm)

<a href="/images/tcp_header1.jpg"><img src="/images/tcp_header1.jpg" alt="The TCP Header" title="tcp_header1" width="300" height="124" class="size-medium wp-image-2819" /></a>

Source: http://www.software-engineer-training.com/wp-content/uploads/2007/12/tcp_header.png

### The Kernel

At a high level, the kernel in an operating system serves as the bridge
between applications and the actual data processing of the hardware the
OS is running on. The kernel manages system resources and abstracts
resources that applications must access.

<a href="/images/kernel.png"><img src="/images/kernel.png" alt="Basic Kernel Representation" title="kernel" width="300" height="237" class="size-medium wp-image-2837" /></a>

### Affected Devices

It is basically all of them save the more recent version. If you've installed a device 
with a JUNOS release version released later then 1/28/09, this issue is already 
corrected. Apparently the original issue and its correction did not conceive of this 
problem as a security vulnerability, and thus the criticality of applying the patch 
was not initially understood until this week.

* JUNOS 10.x  (Removed from the bulletin today, 01/07/09, so assumed to not be affected)
* JUNOS 9.x
* JUNOS 7.x
* JUNOS 8.x

Please note the versions below were removed from the bulletin today, 01/07/09. This 
is likely because, as Matt pointed out below, these 
[are end of life versions](http://www.juniper.net/support/eol/junos.html) of the 
OS (meaning likely still vulnerable if you happen to be running them, but out of 
scope for Juniper because from their standpoint these should already have been 
upgraded).

* JUNOS 6.x
* JUNOS 5.x
* JUNOS 3.x
* JUNOS 4.x

### Juniper's Advice

Juniper references best common practice (BCP) 38, a methodology for reducing the amount 
of bad packets being forwarded by network devices (basically prohibiting packets where 
the originator can't effectively be identified), as a possible mitigating control.

However there is no completely effective workaround available other then upgrading the OS.

### Update

Juniper responded to [the Register](http://www.theregister.co.uk/2010/01/07/juniper_critical_router_bug/) 
as follows: 


> "that the bulletin was one of seven security advisories the company issued 
> under a policy designed to prevent members of the public at large from getting details 
> of the vulnerabilities."
> 
> "Because of Juniper's 'Entitled Disclosure Policy,' only our customers and partners 
> are allowed access to the details of the Security Advisory," 
> 
> - Juniper spokeswoman

Interesting approach, and probably would be better received if vulnerabilities only 
affected those entitled. Unfortunately the networks that run high end Juniper equipment 
serve a great many end users, and thus in this case the general public would probably 
like some informed background. At the point the media is contacting you, it is safe to 
say the "cat is out of the bag". And this is the response from a company that is a 
strong player in the information security appliance space?

The flip side is that the Juniper response to this issue from a technical perspective 
has appeared to be at first glance fairly comprehensive, a PR opportunity if managed 
correctly.

And yes, this is the same firm that feels [this way](http://www.theregister.co.uk/2009/06/30/atm_talk_canceled/) 
when it is they who are discussing the vulnerability of someone else's product: 

> Juniper believes that Jack's research (on ATM vulnerabilities) is important to be 
> presented in a public forum in order to advance the state of security,

We agree with the second Juniper: more education, especially after the problem 
has been corrected, is better.

### Finally

More information will be posted as it becomes available. This was a serious issue 
which appears to have been averted through a coordinated response. Essentially, given 
the core equipment (big Telco routers) running "Big Iron" type Juniper network 
devices, portions of the Internet could have gone black with a successful implementation 
of this exploit. Routers at this level are not patched like your local Windows OS, 
it takes something important to justify an outage. As previously noted, even though 
the code problem itself was identified last year, it appears that the problem was 
not identified as a mechanism for creating a remote exploit until now, raising the 
criticality of patching the issue severely.


