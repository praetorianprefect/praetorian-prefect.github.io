---
layout: page
title: Are borderless networks possiable?
category: 
  - "networking"
  - "security"
tags: 
  - cisco
  - networking
  - security
  - migrated
author: jrossi
header: 
  image_fullwidth: "header-map-night.jpg"
teaser: |
    I attended SC World Congress
    in New York this week and a keynote from Cisco caught my attention: Securing 
    the Cloud: Building the Borderless Network.  I became fixated on the words 
    used over and over by Joel McFarland
    Borderless this, borderless that, borderless everything.  This campaign started 
    to bother me as this was a security conference and a network company was 
    pushing the idea of less borders.  It seemed off, wrong, and incomplete to me.
---


I attended [SC World Congress](http://www.scmagazineus.com/SC-World-Congress-2009/section/886/) 
in New York this week and a keynote from Cisco caught my attention: _Securing 
the Cloud: Building the Borderless Network_.  I became fixated on the words 
used over and over by [Joel McFarland](http://www.scmagazineus.com/Joel-McFarland-senior-manager-Product-Management-Security-Technology-Group-Cisco-Systems/article/149536/). 
Borderless this, borderless that, borderless everything.  This campaign started 
to bother me as this was a security conference and a network company was 
pushing the idea of less borders.  It seemed off, wrong, and incomplete to me.


## Little Bit of History 
 
I am going to quickly cover some of the history of the Internet and how it 
grew borders, but please skip to the highlight of the article if you are 
familiar with this already: [Borderless Networks, What?](#borderless-what)

#### ARPANET ('69-'91)

In the beginning, there was [ARPANET](http://en.wikipedia.org/wiki/ARPANET) 
which was the pioneer in packet switching networks and gave providers the 
choice of which method and hardware for communication it would use.  However, 
the base protocol used for devices to communicate in ARPANET was NCP.  
The NCP  protocol could best be described as a network device driver and 
less as a network transport stack. It did not have any method for end-to-end 
error handling which was seen as a problem, but nothing was done about 
this until 1983. 

In 1983, TCP/IP replaced NCP as the protocol for transport and ARPANET became 
a part of what was to become the Internet.  TCP/IP was a huge improvement 
over NCP in that it accounted for problems on the network and allowed the 
network not to come to a grinding halt when packets were lost.  It also 
achieved the concept of end-to-end connectivity between each host.  This 
meant that as long as two hosts were on the Internet they could reach 
each other by utilizing standard TCP/IP.  This standard framework also 
lead to the growth of many different applications as there was no longer 
any need to make changes to the network to add new applications/protocols.  

#### First Borders ('91-'94)

All the building blocks were in place and  what formed was a large group 
of interconnected networks to share and exchange data. Then the first virus 
and worm hit in 1983 and 1988 respectively.  The 
[morris worm](http://en.wikipedia.org/wiki/Morris_worm) gained a fair amount 
of media attention and in fact prompted the establishment of [CERT
](http://www.cert.org/).  Even in this embryonic stage the vitality of the 
information being shared caused many researchers to begin placing limitations 
on the end-to-end connectivity of their hosts.  Thus began the _'Us'_ 
and _'Them'_ status of the Internet. 

_'Us'_ and _'Them'_ started out simple with a move to keep networks segregated-or put another way, adding a border between the networks.   At first, the borders were nothing more than routers that limited the effects from network _A_ from spilling over into network _B_.  They were effective, but in 1991 [DEC](http://en.wikipedia.org/wiki/Digital_Equipment_Corporation) released the first modern Firewall: SEAL.  This marked the first real security border on the Internet, where all packets were inspected and compared to a set of policy rules before being passed on.  These first security borders were instrumental in providing the trust and assurance in the network that companies and researchers required, speeding the growth of the Internet.  While intrusion was still possible, the bar of entry was raised beyond causal attacks and probes. 

<div class="wp-caption" style="float: right;margin: 5px;margin-left: 42px;margin-right: 21px;"><a title="Figure 1: Us vs. Them" rel="lightbox" href="/images/us-them.png"><img src="/images/us-them.png" border="1" alt="Us vs Them" width="300" height="233" /> </a>
<p class="wp-caption-text"><a title="Figure 1: Us vs. Them" rel="lightbox" href="/images/us-them.png">Figure 1: Us vs. Them</a></p></div>

In 1992, the dominant addressing of hosts was IPv4, where each host is a 
assigned a 32-bit address.  This assignment limited the total number of 
addressable hosts to 4,294,967,296, but, due to reservations and subnetting, 
this could never be fully utilized.  At this time, it was recognized that 
IPv4 limitations would be become a problem in the future, beginning 
the process of creating a new IP protocol with a much higher number 
of addressable hosts. IPv6 was born in 1994, based on a 128-bit 
address for each host.  This would effectively allow every man, 
woman, and child on Earth to be assigned an address many times over.  
As a part of the formation of IPv6, security between networks was 
also taken into account and [IPSec](http://en.wikipedia.org/wiki/IPsec) 
was created as a requirement of the IPv6 protocol.  

IPv6's creation gave the Internet a secure method of communications 
between networks via IPSEC and nearly unlimited address space, but 
IPv6 did not get off the ground quickly.  This was mostly due to the
fact that all devices and operating systems would need to be upgraded to
handle the new protocol, and there was little to no pressure from the 
market to push things forward.   IPSec on the other hand did take off,
as it quickly became the standard method for interconnecting trusted 
networks over an untrusted medium (such as the Internet).

At the same time that IPv6 and IPSec were being developed, another 
group of people began working on an alternate method for dealing 
with the lack of addressable space in IPv4.  
[Network Address Translation (NAT)](http://en.wikipedia.org/wiki/Network_address_translation) 
was published in [RFC1631](http://www.ietf.org/rfc/rfc1631.txt) in 1994 
as a short term solution, while the larger problems were being 
addressed.  NAT became very successful quickly as it allows a very 
large number of hosts to access the larger Internet while using very 
few publicly addressable IP addresses.  As with most things, NAT came
with some trade-offs.  One of the big ones was that hosts no longer 
had complete end-to-end connectivity.  Thus, another border on the 
network was created; in practice firewalls became the dominate NAT 
devices.  Nonetheless, the NAT border would create problems for 
applications developers for years to come.

#### Present ('09)

In 2009, the way Internet runs is really not very different from 
1994;  IPv6 is just now getting underway, NAT is used everywhere, 
and IPSEC still secures networks over an untrusted medium. What 
has changed in a big way is the applications and uses of the Internet.  
Telephone calls commonly use the Internet for transport, on demand 
video is a huge source of traffic, social media networks garner huge
numbers of users, online shopping is an important revenue stream for
companies, and most recently more and more services are being 
hosted elastically on demand via the Internet.


## Borderless Networks. What? {#borderless-what}

Now let's get back to Borderless Networks...

Cisco envisions a global network where you can go any place 
and access any data you could need at anytime.  John Chambers 
detailed the approach on a video at [Cisco.com](http://cisco.com): 




> In terms of what's happening right now, I think the biggest market transition 
> is the shift to a more collaborative world, which is only made possible by 
> what we call an "intelligent, network-centric" world. This network-centric 
> world encompasses the whole range of communication experiences and seamlessly 
> delivers information. Consumers will access voice, the web, e-mail, and 
> video by any of the 14 billion devices that we think will be connected to the 
> internet by 2010, all loaded onto the network. In the very near future, for 
> example, you won't need to hang up your cell phone if you want to switch to 
> a landline; you'll stay connected as you change devices, as long as they're 
> all connected to a network.
> 
> <a href="http://www.cisco.com/survey/exit.html?http://discussionleader.hbsp.com/hbreditors/2008/10/cisco_ceo_john_chambers_on_tea.html">Cisco CEO John Chambers talks about Cisco's collaborative management model</a>


Cisco also has a [Virtual event](http://www.cisco.com/web/solutions/netsys/g2/index.html?POSITION=social+media&COUNTRY_SITE=us&CAMPAIGN=Transformers+Launch&CREATIVE=Borderless+Networks+to+Index&REFERRING_SITE=Twitter) 
on Oct 20th for Borderless Networks, and have been encouraging people to 
register via [twitter](http://twitter.com/CiscoGeeks) and emails for the 
last two weeks. 

<div class="panel panel-default" style="float: right;width: 450px;text-align: left;margin: 5px;margin-left: 20px;">
  <div class="panel-body">

<h4>LUNCH - Securing the cloud: Building the borderless network</h4>

An exploration into the “cloud” revealing the power of choice in email 
security. Learn how to harness all the benefits that the cloud has to 
offer while avoiding common pitfalls for early SaaS solutions. The 
crumbling walls of network perimeters are forcing organizations to architect 
new network designs to address the evolution of borderless networks. <br>
<br>
Attend this session and learn:<br>
- Embracing the change to borderless networks<br>
- Understanding Cisco's next-generation cloud security architecture<br>
- Realizing the power of choice in choosing an email security solution<br>
- Joel McFarland, senior manager in the product management team within the Security Technology Group at Cisco Systems<br>

<p class="wp-caption-text"><a href="http://www.scmagazineus.com/Agenda-Day-1-2009/section/888/">SC World Congress: Agenda Day 1</a></P>
  </div>
</div>

I first learned of the Borderless Networks push during the 
[SC World Congress](http://www.scmagazineus.com/SC-World-Congress-2009/section/886/).  I 
was there to get a preview of Borderless Networks as presented by Joel 
McFarland.  The session description sounded interesting and as it was a 
keynote there was nothing else to pull on my time.

Two co-workers and I attended the session, but being a little late we had to 
make our way to the very front of the room to find seats.  Up front we were able 
to hear and see everything in great detail, but in hindsight this might have 
not been the best place for us. There was no way Joel could have missed the 
looks of skepticism on all three of our faces.  

Joel pushed the Cisco idea of Borderless Networks in many different ways, but 
pointed to the <a title="Figure 2: The iPhone" rel="lightbox" href="/images/iphone_home.gif">iPhone</a> 
as the game changer, the beginning of things to come.  Then iPhone and 
salesforce.com became his prime example of how the mobile sales team are 
almost completely disconnected from the enterprise network.  They access 
leads, manage contacts, input orders, and exchange notes and information all 
without even logging into the corporate network.  At this point, I looked to
my co-workers with a questioning expression and whispered the rhetorical 
question "_No corporate login?_". 

The example Joel used is common for a sales workforce, and is actively 
encouraged in many environments, but this was just something that I have 
always felt was wrong.  In many companies, sales leads are valuable 
information and something that competitors and even other sales people 
would actively try to gain access to.  When all access to this information 
is controlled by an external party you are no longer able to apply your own 
controls. In fact, you are beholden to the policies and procedures of the 
provider.  Joel was one step ahead of me on this.  He pointed out the 
problems that were playing through my head and countered that salesforce.com 
can be made to use a corporation's internal authentication methods (Active 
Directory, RSA Token, etc.).  As such, your internal policies for access 
and removal of access are once again in your control.  I conceded. Joel 
is correct that salesforce.com can be brought into line with one's internal 
security policy, but he does not address the issue of the remote device-the 
iPhone itself.

#### Borderless

Let me come back to the iPhone in a bit, I want to point out another slide 
that came up during this iPhone praise.  In 
<a title="Figure 2: Before & After" href="/images/before-after_borderless.png">Figure 2</a> 
I have created a combined version of the two slides Joel was showing to 
demonstrate the future of networking (I have recreated them from memory, 
but its close enough for this post).  

<div class="wp-caption" style="float: right;margin: 5px;margin-left: 42px;margin-right: 21px;"><a title="Figure 2: Before & After" rel="lightbox" href="/images/before-after_borderless.png"><img src="/images/before-after_borderless.png" border="1" alt="Us vs Them" width="500" height="400" /> </a>
<p class="wp-caption-text"><a title="Figure 2: Before & After" rel="lightbox" href="/images/before-after_borderless.png">Figure 2: Before & After</a></p></div>

In Figure 2, we have the __before__ and __after__ sections. According   
to Joel, currently the __before__ example is a good summary of how most 
enterprises networks allow access into and between their networks. This 
Joel and I agree on.                                                    

As seen in the __before__ section, you have a defined entry point into  
the network from outside, where all external resources gain access.     
This is your border between "_us_" and "_them_". In the examples, both  
the remote home desktop and iPhone access the network and are allowed   
across past the border only if proper authentication and authorization  
have take place. Once completed, the remote device is granted access    
to the resources that are allowed for it to function as an effective    
job tool: access to to internet via internal proxy, access of files in  
the London office, or logging into the salesforce.com website. The key  
thing is that all access flows through this single point of entry.      

By restricting access for remote devices to a single point, we are able
to overcome some technical shortcomings and greatly reduce the vectors
of attack for the network. NAT is required due to the limited number
of publicly addressable addresses. Thus end-to-end connectivity is not
an option for the remote devices. The use of IPSec for transport and
assigning a RFC1918 address to the remote device end of the IPSec tunnel
allows one to overcome the NAT limitations. This gives you remote device
end-to-end connectivity within the enterprise network. By using this
method the network administrators are able to capture and monitor at a
single point all access into and out of the network. NAC, IPS/IDS, and
other methods of monitoring are commonly deployed here.

With the __after__ diagram of Figure 2, we see the future as Cisco/Joel
see it. This is where all resources are able to access all other
resources; also known as complete end-to-end connectivity. Joel did not
say how this was to be achieved, but given the network diagram it's not
hard to surmise that Cisco is planning a big push for IPv6. IPv6 will
allow for this type of network, and will bring down the NAT boundary.
With it the technical limitation of too few addresses for end-to-end
connectivity on the Internet is eliminated and things can get a lot more
complex as we see in the __after__ section of the diagram.

On the __after__ diagram you see end-to-end connectivity to each
resource both inside the network and outside. We have an iPhone going
directly to salesforce.com, directly accessing a file in the London
office, and able to access all the data that it could ever need. What
about limiting access to resources? How do you make sure that a remote
home desktop does not start copying all of the data from the London
office, NYC office, and salesforce.com to a remote site? What if the
desktop is infected with malware? How do you log the activity of the
remote device access? All the questions become much harder when you have
completed end-to-end connectivity, and historically we have learned it
becomes an even larger problem when there are remote devices involved.

All the questions I have asked about the security of the __after__
sections can be answered with products already on the market and in
fact are recommended for use in both networks. The problem becomes the
scale that is needed to protect and defend a network that has complete
end-to-end connectivity. Once again, going back to the __after__
diagram, only taking into account remote device access, the number of
policies that needs to be maintained, protected, and monitored goes from
1 to 4. Now a growth of 400% is big, but almost manageable. If you start
to think about a small enterprise with 20 offices, 2 datacenters, and
200 remote users, the problem of scale is instantly untenable.

IPv6 will solve a lot of problems for networks as the need for NAT
will go away and devices will be able to directly address each other
across networks and boundaries, but as with just about everything there
are side effects. Keeping control of access into and out your network
is the first line of defense and with IPv6 this becomes a policy and
enforcement issue even if it is no longer a technical requirement.


#### The iPhone, Key to the Borderless Network

Joel said he likes his iPhone and from the huge number of videos from
Cisco featuring an iPhone it's safe to assume Cisco does too. During the
keynote Joel pointed out the iPhone a few times in a number examples and
in general with heavy praise. Joel and I agree the iPhone is an amazing
device, an important step forward in mobile computing. After this Joel
and I begin to disagree, namely around one key point: "_The iPhone is a
game changer._" I think that statement needs to add "_for the consumer
market_".

<div class="wp-caption" style="float: left;margin: 5px;margin-left: 5px;margin-right: 21px;"><a title="Figure 3: The iPhone" rel="lightbox" href="/images/iphone_home.gif"><img src="/images/iphone_home.gif" border="1" alt="Us vs Them" width="200" height="330" /> </a>
<p class="wp-caption-text"><a title="Figure 3: The iPhone" rel="lightbox" href="/images/iphone_home.gif">Figure 3: The iPhone</a></p></div>

iPhones are enabling users to use the Internet from
almost anyplace; it's one of the most popular cameras on
[flickr](http://www.flickr.com/cameras/), has a huge list of
applications, and, for some people, a complete replacement for
the traditional computer. While its strong points work well
in the consumer market, in the enterprise markets it's a very
different beast. In fact the strongest points for the iPhone in
the consumer market are security concerns for the enterprise.
Application controls are limited, centralized control is even more
limited, and encryption of the data residing on the devices is a
[problem](http://www.wired.com/gadgetlab/2009/07/iphone-encryption/) on
the most fully featured phone to date.

Devices like the iPhone should be thought of less as a phone and more as
a laptop. With that comes all the same protections and controls that we
use to mitigate risk on an enterprise laptop. Here is a quick list of
what I expect from a laptop and by extension from an iPhone for it to
become a viable remote access device in the enterprise environment:

* Virus and Malware software with centralized reporting
* Secure communications for the device; both internal resources and the ability to define policies
* Strong Data Encryption on the device
* Ability to do remote kill of device
* Application installation and run controls
* Web Filter/Proxy controls
* Access controls, password complexity settings and password failure data destruction

Some of the areas listed are available on the iPhone, but none of them
are near complete and ready for everyday use in an enterprise. [Research
In Motion](http://www.rim.com/) (RIM) dominates the enterprise market
for the reasons I have listed here. RIM via the BlackBerry Enterprise
Server (BES) gives the enterprise complete control of every device that
connects via a centralized management station. BES also does network
traffic correctly in that all devices came back to the BES at a single
point of entry into the enterprise. This allows an enterprise to place
additional control directly attached to the BES and not with multiple
devices all over the network. RIM's BES product represents the minimum
level of security that should be expected for remote access of phone
like devices. I would go so far as to say it should be the starting
standard for how remote access devices should behave.

The iPhone might be the start of things to come, but in no way is it
even close to ready for the enterprise market.

## Why?

Cisco's push with Borderless Networks is either something that they
haven't completely vetted from a security perspective or the security
strategy isn't completely explained in the marketing. The huge increase
in the number of points needing protection, the corresponding increase
in the policy and management, and management data flow and access
controls are areas that need addressing. These are problems we still
having troubles controlling with our current network deployments. Unless
Cisco has a magic bullet coming out of their research and development
departments, I don't see how this move to Borderless Networks is even
possible.


