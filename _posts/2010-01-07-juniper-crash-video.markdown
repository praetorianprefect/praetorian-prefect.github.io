---
layout: post
title: JUNOS (Juniper) Kernel Crash Video
date: 2010-01-07 
categories: 
  - security
tags: 
  - juniper
  - tcp
  - option-101
  - networking
  - crash
  - junos
  - migrated
series:
  - "Praetorian Prefect"
slug: juniper-kernel-crash-video
aliases:
  - /juniper-kernel-crash-video.html
author: Jeremy Rossi
teaser: |
    We have noted some interesting responses since our post yesterday
    detailing the information in Juniper bulletin PSN-2010-01-623 and our thoughts 
    on its somewhat understated effect. Since our post yesterday, the bulletin 
    has been updated, becoming more specific about the versions affected (basically 
    excluding JUNOS version 10.x and versions no longer supported by Juniper). 
---


We have noted some interesting responses since [our post yesterday](http://praetorianprefect.com/archives/2010/01/junos-juniper-flaw-exposes-core-routers-to-kernal-crash/) 
detailing the information in Juniper bulletin PSN-2010-01-623 and our thoughts 
on its somewhat understated effect. Since our post yesterday, the bulletin 
has been updated, becoming more specific about the versions affected (basically 
excluding JUNOS version 10.x and versions no longer supported by Juniper). 
We've been quoted here and there saying that [the potential worst case scenario](http://www.theregister.co.uk/2010/01/07/juniper_critical_router_bug/) 
with this flaw could have been widespread Internet outages (not overstatement in our opinion), 
and that such a simple attack that escapes filtering and 
[can reboot high end routers is a big deal](http://www.computerworld.com/s/article/9143342/Juniper_patches_router_crashing_bug). 
We have tested sending all 256 permutations of the Options field in the TCP header 
to a vulnerable Juniper router operating system, found the correct value, and 
reproduced the kernel crash, which is demonstrated in the video below.

[SCAPY](http://www.secdev.org/projects/scapy/) was used to send the packets used in the test.

<object width="700" height="674" classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,40,0"><param name="allowfullscreen" value="true" /><param name="allowscriptaccess" value="always" /><param name="src" value="http://vimeo.com/moogaloop.swf?clip_id=8606222&amp;server=vimeo.com&amp;show_title=0&amp;show_byline=0&amp;show_portrait=0&amp;color=00ADEF&amp;fullscreen=1" /><embed width="700" height="674" type="application/x-shockwave-flash" src="http://vimeo.com/moogaloop.swf?clip_id=8606222&amp;server=vimeo.com&amp;show_title=0&amp;show_byline=0&amp;show_portrait=0&amp;color=00ADEF&amp;fullscreen=1" allowfullscreen="true" allowscriptaccess="always" /></object>


### Responses to Our Original Post

#### The Bizarre
We've seen kind of off the wall responses, like this one:

> It seems like this isn't as major as they say. Sure it's a kernel crash, but 
> it requires a packet to be sent to a listening port. I doubt any core routers 
> have any ports open to the public internet at all.

In order for a router to function as a router, some TCP ports must be open. 
The BGP port will be open on a core router. So yes, a core router will not have 
ports open to the public Internet. The BGP port however will be open to neighbors, 
and a packet that cannot be filtered negates ACL rules preventing access by 
anyone but neighbors. At a high level, that is how high end equipment is affected.

#### The Official

We saw the response from Juniper we talked about yesterday repeated again today, 
which continues to leave something to be desired: A Juniper spokeswoman declined 
to provide more technical details on the issue, saying that the company only 
passes on this information to customers and partners. The advisory was one of 
seven issued recently by the company, she said via e-mail.

Yes, there were seven advisories. Six were somewhat less interesting than one of them:

* PSN-2010-01-627 - RPD cores when injected with malformed PIM messages - (As it is not commonly used over the Internet, this issue is confined to the organizations that are running PIM internally)
* PSN-2010-01-626 - BGP Malformed AS-4 Byte Transitive Attributes Drop BGP Sessions - (If you are running an affected version (there aren't many), upgrade ASAP.)
* PSN-2010-01-625 - Invalid RSVP packet causes RPD process busy loop and router becomes unresponsive - (RSVP is used almost excursively inside a services providers network as part of a larger MPLS Traffic Engineering solution. Due to the use of MPLS VPN's, RSVP in this environment is not exposed to transit traffic or from within the VPN's. The exposure of this is much lower.)
* PSN-2010-01-624 - Unauthorized user can obtain root access using cli - (Any access escalation issue is a big problem, but in this case for routers, if someone else is able to login and get console access you have other problems that need to be addressed.)
* PSN-2010-01-623 - JUNOS kernel cores when it receives an crafted TCP option. - (Not so good.)
* PSN-2010-01-622 - as-path-prepend and specific length AS_PATH we can cause a Juniper to send corrupted update packets to eBGP neighbors - (BGP as-path-prepend router level configuration that can be corrected by making changes to the config.)
* PSN-2010-01-621 Crafted RSVP Path Object Overloads the RPD Process - (RSVP is used almost exclusively inside a service providers network as part of a larger MPLS Traffic Engineering solution. Due to the use of MPLS VPN's RSVP in this environment is not exposed to transit traffic or from traffic within the VPN's. The exposure of this problem is lower.)

#### Unofficial, but from Juniper Anyway

We received [a response from Matt at Juniper in the comment section](http://praetorianprefect.com/archives/2010/01/junos-juniper-flaw-exposes-core-routers-to-kernal-crash/#comments) 
of the original post, which we appreciated. He tightened the versions affected 
information, by noting the mistake in the original Juniper bulletin that stated 
version 10.x was affected.

Again, thanks for the update Matt.

#### Another Unofficial, but from Juniper Anyway

JuniperPhilly responds in the [comments of the Register article](http://forums.theregister.co.uk/forum/1/2010/01/07/juniper_critical_router_bug/) as follows:


> it's probably not as bad as you might think- All Junos software releases built 
> on or after January 28, 2009 have fixed this specific issue. In short, we 
> fixed this particular problem about 350 days ago.
> ....
> Disclaimer: I work for Juniper as a Systems Engineer.

Well, sort of. The criticality of the defect was certainly reclassified, so the fix 
made a while back actually seems divorced from the discovery that this problem leads 
to a kernel crash based on a remote exploit. The Juniper advisory itself reads this 
way, suggesting that the fix was made without knowing that it was a fix for a remote 
exploit. This is not that uncommon, problems are fixed for one reason, without ever 
knowing there was an even better reason for correcting it.

But routers, especially high capacity ones, are only patched for serious reasons. So 
a defect identified but not reported in the same way back in January 2009 does not carry 
the affect of releasing a bulletin labeled critical yesterday. The second makes people 
maintaining those routers move, as the example below shows.

[Qwest](http://news.qwest.com/company), like other backbone providers, doesn't have 
unannounced outages for unspecified security concerns over "not as bad as you might think" 
issues:


> Date: 2010-01-07 10:04:08 GMT (15 hours and 1 minute ago)
> We just had a qwest outage of about 2 mins at 1:41am pst. When I called
> to report it I was told it was a 200+ emergency software upgrade due to
> a security concern, and that we will get a notice later after the fact.
> Normally we get notices in advance, even for software upgrades due to
> security or other important issues, so I am curious if other qwest
> customers had the same experience and wether this is how it's going to
> be from here on in? The affected platform was juniper and I'd love to
> know the specfic case being addressed here.
> 
> Mike-

Source: [http://thread.gmane.org/gmane.org.operators.nanog/71244](http://thread.gmane.org/gmane.org.operators.nanog/71244)

This thread actually produced interesting responses regarding how the actual 
notification was published after the outage:

* My QWest account manager called three different people at my business 7hrs before 
  the maintenance. Also mentioned the Juniper Security Advisories. - Joe
* We also got email notifications about 'emergency maintenance' on our Qwest circuits, 
  from their notice: Reason For Maintenance: EMERGENCY MAINTENANCE TO IMPLEMENT A 
  SOFTWARE PATCH FOR NETWORK RELIABILITY - Ken
* Yeah, they refused to notify due to security concerns from what they told me 
  last night. Notification was performed after maintenance was complete. -Jack
* Same thing for us in Minnesota. Brief outage and emergency outage notification 
  came after the outage. -Dylan
* Notices were left at the discretion of Qwest account teams. There was no mass 
  notification. -Jason

The thread link above contains this and the rest of this particular discussion.

#### The Newsgroups

We were told the problem wasn't corroborated by discussions in newsgroups. It started showing up today:

* [qwest outage no notice](http://thread.gmane.org/gmane.org.operators.nanog/71244)
* [JUNOS vulnerability with malformed TCP packets](http://thread.gmane.org/gmane.network.nsp.juniper/15350)
* [vulnerability fix not available for 8.5 ?](http://thread.gmane.org/gmane.network.nsp.juniper/15366)
* [JUNOS vulnerability with malformed TCP packets](http://thread.gmane.org/gmane.network.nsp.juniper/15356)

### Yeah but Cisco makes the Core Routers

Sigh...

Not to become public relations for Juniper, but:

The innovations listed above, as well as many others, have helped the T Series become 
the industry's most widely deployed core routing family. Juniper has shipped over 
5000 T Series to more than 220 customers around the world — including more than 
500 T1600s in just over a year of availability. According to Synergy Research, in 
the past five years, Juniper's share of the core routing market has grown by 44 
percent — with the company gaining 11 points of share as others have seen share 
declines.

Source: [http://www.juniper.net/us/en/company/press-center/press-releases/2009/pr_2009_06_08-09_00.html](http://www.juniper.net/us/en/company/press-center/press-releases/2009/pr_2009_06_08-09_00.html)

And the following line from the same press release:

All of these platforms are powered by JUNOS® Software, a single operating system integrating 
routing, switching, security and network services from Juniper Networks.

### What about Anti-spoofing and egress filtering

> (Comments From: ANTON DELPORT)
> 
> One thing that will also be required for a successful attacked would be spoofed IP packets.
> Keep in mind that most ISP follow the best practice guidelines and implement ACL and
> anti-spoofing. So yes, the router will listen to BGP port but only for a small range
> of prefixes. If the source address (and destination) is not correct, the packet will
> be dropped in hardware before it can do any damage.

Anti-spoofing and egress filtering as recommended by BCP 38 is to help mitigate this issue 
for routers that are not at the edge. It does nothing to help the edge routers themselves. 
Example:

* Service provider Alice peers with service provider Bob in NYC.
* Alice's edge router (`10.10.10.1/30`) exchanges routes with Bob's edge router (`10.10.10.2/30`) via BGP
* Bad actor Charlie sends a JunOS rebooting packets from inside Alice's network to `10.10.10.2`
* The best path from with in Alice's network to reaching `10.10.10.2` will most likely be the peering connection in NYC.
* The packet will *NOT* be stopped within Alice's network as it has a valid return and destination address.
* Bob's edge router is *NOT* able to filter any of the JunOS rebooting packets due to ACL's not having any effect on this issue.
* Even if the bad actor Charlie is several networks away from Bob, should his packet pass through Alice's network, it will hit Bob's edge and cause the same harm.

The reason why this issue is real is that I can identify border networks simply with `traceroute`, 
and I know that BGP is used to exchange routes. Given this information there is nothing to 
protect providers if they are running an affected version of the software at the edge of their network.

### Finally

So people are attaching viewpoints to this problem that don't entirely make sense. A high end 
router is not the same as your local Microsoft Windows OS, it doesn't get updated every month 
following Tuesday, it gets updated when a network administrator determines there is a problem 
severe enough to warrant an outage to make the patch update. Many of the "big iron" routers 
that would have been affected had this been out in the wild (which as far as we know its not 
yet) were not patched as of Monday, and from all appearances were patched as of late Tuesday.

Juniper is a major player in the high end router market, it is not a one player market. 
If an unpatched Juniper router were hit with this packet, it would crash.

But let's walk through a thought experiment for the "this wouldn't have been a big deal 
if uncorrected" crowd:

Watch the video above, the OS reboot takes a while on a virtual machine (big routers take 
longer). Imagine a bot net being rented to run the program that was developed for the video 
above at a certain time (say midnight). Conceive of the bad actor identifying boundary 
routers between service providers (traceroute), and sending the crafted packet to the BGP 
port of both side's IP addresses, rebooting boxes, and severing BGP connections. Even 
after reboot, the effects are magnified as a BGP convergence happens globally.

You can rent a decent size botnet on the Internet right now if you like. The program 
above that found the right option to send took a couple hours to write (on and off 
with other things going on), the actual option field that causes the problem identified 
fairly quickly after that. The second program that sends the packet is just a small 
python script.

This hypothetical scenario would have been a long day on the old Intertubes. I'm 
sure there are details to be worked out (if you crash enough gateways, can you continue 
the attack?), but you get the idea.

So let's be realistic as we go into the automatic "nothing is ever really a big issue, 
everything is FUD" reactive mode that so often follows news in information security. 
Remote exploits are still bad. Ones that cause kernel crashes are still bad. Remote 
exploits that cause kernel crashes in one of the most widely used network operating 
systems in the world are bad. Identifying security issues that are critical, responding 
to them appropriately, sending out bulletins with appropriate CVSS ratings, and 
avoiding big potential problems like this, are good. We can't call it a total win 
(its not hard to find the option value, and so this could enter the wild shortly), 
but it looks from the outside like large providers have taken preventative steps to 
be prepared.

And if anyone else noticed Twitter seemed to have its own blackout, of Juniper personnel, 
as none of them have been tweeting a whole lot this week.

