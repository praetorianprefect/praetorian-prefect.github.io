---
layout: page
title: "VRF is the new Black: How I Learned to Stop Worrying and Love the Complexity"
categories: 
  - networking
tags: 
  - cisco
  - juniper
  - junos
  - ios
  - secreenos
  - vrf 
  - networking
  - security
author: jrossi
teaser:  |
    Breaking up your network is good, we all know this, and VLANs have
    traditionally been used to segment a network to help with maintenance,
    management, and security; but, they are not the only game in town and often the
    wrong place to break your network into smaller and more efficient pieces. VPN
    Routing and Forwarding (VRF) can do the same for layer 3 infrastructure that
    VLANs do for layer 2. By allowing you to create and manage separate routing
    tables within a single physical router, they truly bring virtualization and
    segmentation to all points on your network. As with any technology that adds
    layers, complexity can become a problem, but you already know this.
---


Breaking up your network *"is good,"* we all know this, and VLANs have
traditionally been used to segment a network to help with maintenance,
management, and security; but, they are not the only game in town and often the
wrong place to break your network into smaller and more efficient pieces. VPN
Routing and Forwarding (VRF) can do the same for layer 3 infrastructure that
VLANs do for layer 2. By allowing you to create and manage separate routing
tables within a single physical router, they truly bring virtualization and
segmentation to all points on your network. As with any technology that adds
layers, complexity can become a problem, but you already know this.

## Table of Contents

* [Virtual Routing and Forwarding (VRF)](#vrf-intro)
* [VRF Lite Setup](#vrf-setup)
    * [Juniper JunOS](#setup-junos)
    * [Cisco IOS](#setup-ios)
    * [Juniper ScreenOS](#setup-screenos)

## Virtual Routing and Forwarding (VRF) {#vrf-intro}

> "It's incredibly obvious, isn't it? A foreign substance is introduced into our
> precious bodily fluids without the knowledge of the individual, and certainly
> without any choice."
>
> `Gen Jack D. Ripper`

Virtual routing and forwarding (VRF) is a technology included in network routers
that allows multiple instances of a routing table to exist in a single router
all while working simultaneously.

Their are two types of VRFs: *"VRF"* and *"VRF Lite."*

VRF Lite is just a subset of VRF without all the protocols used for creation of
VPNs between routers, namely MPLS. VRFs are very common in service providers
networks and at some point nearly all internet traffic passes through a VRF or
two.

VRF Lite allows for interfaces on a physical router to belong to a routing instance.  This routing instance has its own forwarding table, ARP entries, and everything else needed to make a forwarding decision.  It can simply be thought of as a router within a router (*<a title="Routers in router" rel="lightbox" href="/images/router-in-router.png"> Figure 1</a>*).

<div class="wp-caption" style="float: right;margin: 5px;margin-left: 60px;margin-right: 21px;"><a title="Routers in router" rel="lightbox" href="/images/router-in-router.png"> <img src="/images/router-in-router.png" border="0" alt="router in router.png" width="200" height="135" />
<p class="wp-caption-text">Figure 1: Routers within Router</p>

</a></div>

This structure makes VRFs useful for many applications and as a solution to
quite a few tough network design issues. It can be used to improve the network
in the following ways:

* [Segmentation](#vrf-intro-seg)
* [Management and Control](#vrf-intro-mgmt)
* [Security](#vrf-intro-sec)

### Segmentation {#vrf-intro-seg}

Layer 2 segmentation based on VLANs and firewalls is showing strains and being
pushed beyond reasonableness when it comes to how a network architecture should
be built. A good example of this is 10 Gig and 1 Gig Ethernet MANs[^1] that span
multiple buildings and datacenters into a single campus. An overview of a large
campus network can been seen in <a title="Large MAN Overview" rel="lightbox"
href="/images/man-example.png">Figure 2</a>.

In our example network, creating wired guest access would require the use
of firewalls in each building or extending VLANs between buildings to the
centralized firewalls in the datecenter. Both options have downsides that VRFs
would be better at solving.

<div class="wp-caption" style="float: right;margin: 5px;margin-left: 60px;margin-right: 21px;"><a title="Figure 1: Large MAN Overview" rel="lightbox" href="/images/man-example.png"><img src="/images/man-example.png" border="1" alt="MAN Network Diagram" width="200" height="204" /> </a>
<p class="wp-caption-text"><a title="Figure 1: Large MAN Overview" rel="lightbox" href="/images/man-example.png">Figure 2: Large MAN Overview</a></p>

</div>

In the case of extending VLANs between buildings this would have the campus
network design rely on Spanning Tree and layer 2 protocols to provide a
loop-free environment. In the case of a large network such as our example, this
could lead to long failover times during hardware failure, while also not making
full use of all available network bandwidth.

The use of firewalls mitigates most of the network utilization and failure times
by making use of layer 3 routed campus design, but this comes at a large cost.
Namely, the cost is incurred in maintenance and raw hardware costs for large
firewalls that are able to deal with 10 Gig and 1 Gig ethernet line rates. The
use of access-lists are often supplemented for firewalls to reduce costs, but
this approach is fraught with issues and access-lists are never reviewed often
enough.

A VRF based solution for a wired guest network on a large campus would
allow guest traffic to be routed to the firewalls in the datacenters via
routing policy while still being segmented away from production traffic. By
leveraging VRFs none of the aforementioned compromises are required to keep
this separation. The production network is able to fully utilize all available
links and not relay on spanning tree protocol between sites for a loop free
environment.

### Management and Control {#vrf-intro-mgmt}

For managing devices on a network, there is a need for out of band (OOB)
connections. There really is no other sure-fire way of gaining access during
a truly catastrophic event other than this tried and true modem/console
connection. But for the daily running and maintenance of the network, OOB just
can not keep up with the needs of daily maintenance and the amount of traffic
generated by NetFlow, logging, ftp/tftp backups, and scp (secure copy) of new
images. To complete these high bandwidth functions, most companies I have seen
and worked with just resort to using the network that servers and even desktops
traffic utilize. This traffic in many cases is highly sensitive and really
should not be available to anyone outside of authorized users.

VRFs can help to move this traffic out of the primary network and into a second
network that only services management functions and has no direct access to
the Internet, desktops, or other uncontrolled resources. In fact, Cisco is now
adding VRF management ports to some of their newer devices[^4]. The use of ACL's
and other forms of control and logging are still needed, but they become simpler
to keep updated and are normally far less complicated when production traffic is
neither expected nor allowed.

### Security {#vrf-intro-sec}

> "I... I don't know exactly how to put this, sir, but are you aware of what a serious breach of security that would be?
> I mean, he'll see everything, he'll... he'll see the Big Board!"
>
> `Gen "Buck" Turgidson`

VRFs allow for complete separation of different routing instances from one
another. This simple and effective concept of hiding networks from each other
and limiting the ability of devices from interacting outside of defined
boundaries creates a more secure network. A good example of this would be a
voice network within a campus. In general, there is very little reason for VoIP
end points to speak to anything other than the voice gateway and each other.
Moving of voice traffic to a VRF allows for gateways to still interact and even
direct device-to-device interconnection, while greatly reducing the attack
vectors.

VRFs do increase the surface area of your network devices due to the increased
number of addressable interfaces on each hardware device. But I would counter
this with the fact that the network is divided into more domain specific
networks. The ACL and protection measures required become much simpler to
implement and keep up to date. A good and simple example of this would be to
just block all management functions for anything outside of the management VRF.

## VRF Lite Setup {#vrf-setup}

VRF Lite is supported on most modern network hardware, but I personally have not used them 
outside of [Juniper JunOS](http://juniper.net/products/junos/), 
[Juniper ScreenOS](http://www.juniper.net/techpubs/software/screenos/screenos6.1.0/index.html), 
and Cisco [IOS](http://cisco.com/go/ios).  Each Platform/Company has it's own 
naming[^3] convention for the this feature, but the concept is the same in each.

> "Gentlemen, you can't fight in here! This is the War Room."
>
> `Pres Merkin Muffley`

* [Setup on Juniper JunOS](#setup-junos)
* [Setup on Cisco IOS](#setup-ios)
* [Setup on Juniper ScreenOS](#setup-screenos)

### VRF Lite Setup on Juniper JunOS {#setup-junos}


First we need to setup some basic interfaces for later use. We will not be
assigning them an IP address as I do not want to pollute the global routing
table[^2]. We will be using VLANs on ethernet interfaces to break up the router
`junos-1` into three virtual routers.

Enable VLAN tagging on the interfaces and create some sub interfaces.

~~~ bash
set interfaces fe-0/0/0 vlan-tagging
set interfaces fe-0/0/0 unit 100 vlan-id 100
set interfaces fe-0/0/0 unit 100 description "Untrust"
set interfaces fe-0/0/0 unit 200 vlan-id 200
set interfaces fe-0/0/0 unit 200 description "Trust"
set interfaces fe-0/0/0 unit 300 vlan-id 300
set interfaces fe-0/0/0 unit 300 description "DMZ"
set interfaces fe-0/0/0 unit 400 vlan-id 400
set interfaces fe-0/0/0 unit 400 description "Trust"
~~~

The verify the results and commit the changes.

~~~ bash
[edit]
jrossi@junos-1# show interfaces
fe-0/0/0 {
	vlan-tagging;
	unit 100 {
		description Untrust;
		vlan-id 100;
	}
	unit 200 {
		description Trust;
		vlan-id 200;
	}
	unit 300 {
		description DMZ;
		vlan-id 300;
	}
	unit 400 {
		description Trust;
		vlan-id 400;
	}
}

[edit]
jrossi@junos-1# commit
commit complete

~~~ 

Now let's create three new routing-instances: Trust, Untrust, and DMZ. The
`instance-type ` supports quite a few option types on JunOS, but to to create a
VRF Lite instance we just need to use `virtual-router`. We also need to assign
interfaces to each newly created instance. This is very different than in Cisco
IOS in that one configures VRF in the interface configuration hierarchy.

~~~ bash
show routing-instances
set routing-instances Trust instance-type virtual-router
set routing-instances Trust interface fe-0/0/0.200
set routing-instances Trust interface fe-0/0/0.400
set routing-instances Untrust instance-type virtual-router
set routing-instances Untrust interface fe-0/0/0.100
set routing-instances DMZ instance-type virtual-router
set routing-instances DMZ interface fe-0/0/0.300

~~~ 
View the results and commit the change.
~~~ bash
[edit]
jrossi@junos-1# show routing-instances
Trust {
	instance-type virtual-router;
	interface fe-0/0/0.200;
	interface fe-0/0/0.400;
}
Untrust {
	instance-type virtual-router;
	interface fe-0/0/0.100;
}
DMZ {
	instance-type virtual-router;
	interface fe-0/0/0.300;
}

[edit]
jrossi@junos-1# commit
commit complete

~~~ 

Now, we have the interfaces configured and set up without addresses. If we look
at the routing table nothing shows up because we have not enabled any interface
families. Once we add address to the `family inet` interface configuration, the
routing table will begin to take shape.

~~~ bash
jrossi@junos-1# run show route

inet.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 03:47:47
                    > to 10.4.37.1 via fe-0/0/1.0
10.4.37.0/24       *[Direct/0] 1d 19:35:26
                    > via fe-0/0/1.0
10.4.37.9/32       *[Local/0] 1d 19:35:26
                      Local via fe-0/0/1.0
192.168.5.0/24     *[Direct/0] 1d 13:13:18
                    > via fe-0/0/1.0
192.168.5.123/32   *[Local/0] 1d 13:13:18
                      Local via fe-0/0/1.0
224.0.0.5/32       *[OSPF/10] 1d 12:50:00, metric 1
                     MultiRecv

__juniper_private2__.inet.0: 1 destinations, 1 routes (0 active, 0 holddown, 1 hidden)
~~~ 

Let's add some interface `family inet` addresses. I am going to use overlapping
address ranges to show that when VRF is used they do not interfere with each
other.

~~~ bash
set interfaces fe-0/0/0 unit 100 family inet address 10.10.10.1/24
set interfaces fe-0/0/0 unit 200 family inet address 172.16.10.1/24
set interfaces fe-0/0/0 unit 300 family inet address 10.10.10.1/24
set interfaces fe-0/0/0 unit 400 family inet address 192.168.10.1/24
~~~ 

Now let's verify the changes and commit them.

~~~ bash
jrossi@junos-1# show interfaces fe-0/0/0 
vlan-tagging;
unit 100 {
    description Untrust;
    vlan-id 100;
    family inet {
        address 10.10.10.1/24;
    }
}
unit 200 {
    description Trust;
    vlan-id 200;
    family inet {
        address 172.16.10.1/24;
    }
}
unit 300 {
    description DMZ;
    vlan-id 300;
    family inet {
        address 10.10.10.1/24;
    }
}
unit 400 {
    description Trust;
    vlan-id 400;
    family inet {
        address 192.168.10.1/24;
    }
}

[edit]
jrossi@junos-1# commit 
commit complete
~~~ 

When we look into the routing you see much more information and can even see the
different routing instances. The global routing table `inet.0` is the default
table your would normally work with. Further down the list you see `DMZ.inet.0`,
`Trust.inet.0`, and `Untrust.inet.0`; they are the newly created VRF Lite
routing instances.

~~~ bash
[edit]
jrossi@junos-1# run show route 

inet.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 04:27:06
                    > to 10.4.37.1 via fe-0/0/1.0
10.4.37.0/24       *[Direct/0] 1d 20:14:45
                    > via fe-0/0/1.0
10.4.37.9/32       *[Local/0] 1d 20:14:45
                      Local via fe-0/0/1.0
192.168.5.0/24     *[Direct/0] 1d 13:52:37
                    > via fe-0/0/1.0
192.168.5.123/32   *[Local/0] 1d 13:52:37
                      Local via fe-0/0/1.0
224.0.0.5/32       *[OSPF/10] 1d 13:29:19, metric 1
                      MultiRecv

__juniper_private2__.inet.0: 1 destinations, 1 routes (0 active, 0 holddown, 1 hidden)

DMZ.inet.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.0/24      *[Direct/0] 00:00:06
                    > via fe-0/0/0.300
10.10.10.1/32      *[Local/0] 00:00:06
                      Local via fe-0/0/0.300

Trust.inet.0: 4 destinations, 4 routes (4 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.10.0/24     *[Direct/0] 00:00:18
                    > via fe-0/0/0.200
172.16.10.1/32     *[Local/0] 00:00:18
                      Local via fe-0/0/0.200
192.168.10.0/24    *[Direct/0] 00:00:06
                    > via fe-0/0/0.400
192.168.10.1/32    *[Local/0] 00:00:06
                      Local via fe-0/0/0.400

Untrust.inet.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.0/24      *[Direct/0] 00:03:26
                    > via fe-0/0/0.100
10.10.10.1/32      *[Local/0] 00:03:26
                      Local via fe-0/0/0.100



~~~ 

While having interfaces with addresses and different routing tables is cool and
all, this does next to nothing as there is no real routing going on so let's add
some.

Start out by adding a default route to the `Trust` VRF lite configuration. The
commands to perform this are almost exactly the same for the global routing
table. The only difference is that you start under the `routing-instances `
configuration hierarchy. This also applies for routing protocols.

~~~ bash
set routing-instances Trust routing-options static route 0.0.0.0/0 next-hop 192.168.10.2
~~~ 

Now let's verify our configuration and commit the change.

~~~ bash
[edit]
jrossi@junos-1# show routing-instances Trust 
instance-type virtual-router;
interface fe-0/0/0.200;
interface fe-0/0/0.400;
routing-options {
    static {
        route 0.0.0.0/0 next-hop 192.168.10.2;
    }
}

[edit]
jrossi@junos-1# commit 
commit complete
~~~ 

Now let's take a look at the `Trust.inet.0` routing table. This time we are
going limit our show route command to just the `Trust` table.


~~~ bash
[edit]
jrossi@junos-1# run show route table Trust 

Trust.inet.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 00:36:26
                    > to 192.168.10.2 via fe-0/0/0.400
172.16.10.0/24     *[Direct/0] 00:00:18
                    > via fe-0/0/0.200
172.16.10.1/32     *[Local/0] 00:00:18
                      Local via fe-0/0/0.200
192.168.10.0/24    *[Direct/0] 00:56:56
                    > via fe-0/0/0.400
192.168.10.1/32    *[Local/0] 00:56:56
                      Local via fe-0/0/0.400
~~~ 

### VRF Lite Setup on Cisco IOS {#setup-ios}


Cisco IOS is used here and it's very new and buggy 12.4T(22), but as this is
what I installed to test other features of IOS, I figured it would not be a
problem for this write up. It should also be more than adequate for VRF Lite.
Please note that there are a large number of extra interfaces and features
configured on this router as I do lots of playing around with IOS on this
device.

Just like in the JunOS Example, we are going to create some sub-interfaces to
start off with.


~~~ bash
ios-1(config)#int gi0/0
ios-1(config-if)#no shut
ios-1(config-i)#int gi0/0.100
ios-1(config-subif)#description Untrust
ios-1(config-subif)#encapsulation dot1Q 100
ios-1(config-subif)#int gi0/0.200
ios-1(config-subif)#description Trust
ios-1(config-subif)#encapsulation dot1Q 200
ios-1(config-subif)#int gi0/0.300
ios-1(config-subif)#description DMZ
ios-1(config-subif)#encapsulation dot1Q 300
ios-1(config-subif)#int gi0/0.400
ios-1(config-subif)#description Trust
ios-1(config-subif)#encapsulation dot1Q 400
~~~ 

Just a quick peek to see that things are as we expect them.

~~~ bash
ios-1(config-subif)#do show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES NVRAM  up                    up
GigabitEthernet0/0.100     unassigned      YES unset  up                    up
GigabitEthernet0/0.200     unassigned      YES unset  up                    up
GigabitEthernet0/0.300     unassigned      YES unset  up                    up
GigabitEthernet0/0.400     unassigned      YES unset  up                    up
GigabitEthernet0/1         1.1.1.1         YES NVRAM  up                    up
FastEthernet0/3/0          unassigned      YES unset  down                  down
FastEthernet0/3/1          unassigned      YES unset  up                    down
FastEthernet0/3/2          unassigned      YES unset  up                    down
FastEthernet0/3/3          unassigned      YES unset  up                    down
ATM0/1/0                   unassigned      YES NVRAM  administratively down down
ATM0/1/0.1                 unassigned      YES unset  administratively down down
Dot11Radio0/2/0            unassigned      YES NVRAM  up                    up
Dot11Radio0/2/0.1          192.168.128.1   YES NVRAM  up                    up
Dot11Radio0/2/0.3          192.168.11.1    YES NVRAM  up                    up
Dot11Radio0/2/0.4          192.168.4.1     YES NVRAM  up                    up
Dot11Radio0/2/0.5          unassigned      YES unset  up                    up
Dot11Radio0/2/0.10         192.168.10.1    YES NVRAM  up                    up
Dot11Radio0/2/1            unassigned      YES NVRAM  administratively down down
Vlan1                      unassigned      YES NVRAM  up                    down
Vlan3                      192.168.3.1     YES NVRAM  up                    down
Vlan5                      unassigned      YES NVRAM  up                    down
Vlan20                     192.168.20.1    YES NVRAM  up                    down
NVI0                       192.168.1.1     YES unset  up                    up
SSLVPN-VIF0                unassigned      NO  unset  up                    up
BVI3                       192.168.5.1     YES NVRAM  up                    up
Loopback1                  192.168.1.1     YES NVRAM  up                    up
Loopback69                 192.168.69.1    YES NVRAM  up                    up
Loopback100                unassigned      YES NVRAM  up                    up
Loopback666                10.10.10.2      YES NVRAM  up                    up
Tunnel255                  192.168.255.2   YES NVRAM  up                    up

ios-1(config-subif)#do show int desc
Interface                      Status         Protocol Description
Gi0/0                          up             up
Gi0/0.100                      up             up       Untrust
Gi0/0.200                      up             up       Trust
Gi0/0.300                      up             up       DMZ
Gi0/0.400                      up             up       Trust
Gi0/1                          up             up
Fa0/3/0                        down           down
Fa0/3/1                        up             down
Fa0/3/2                        up             down
Fa0/3/3                        up             down
AT0/1/0                        admin down     down
AT0/1/0.1                      admin down     down
Do0/2/0                        up             up
Do0/2/0.1                      up             up
Do0/2/0.3                      up             up
Do0/2/0.4                      up             up
Do0/2/0.5                      up             up
Do0/2/0.10                     up             up
Do0/2/1                        admin down     down
Vl1                            up             down
Vl3                            up             down
Vl5                            up             down
Vl20                           up             down
NV0                            up             up
SS0                            up             up
BV3                            up             up
Lo1                            up             up
Lo69                           up             up       for webvpn
Lo100                          up             up
Lo666                          up             up
Tu255                          up             up

~~~ 

Much like in the JunOS configuration we will now create three new routing
instances (VRF Lite).

~~~ bash
ios-1(config)#ip vrf
ios-1(config)#ip vrf Untrust
ios-1(config-vrf)#ip vrf Untrust
ios-1(config-vrf)#description Scary wild wild west
ios-1(config-vrf)#ip vrf Trust
ios-1(config-vrf)#ip vrf DMZ
~~~ 

> I don't give a hoot in Hell how you do it, you just get me to the Primary, ya hear!
>
> `Major T. J. "King" Kong`

Now let's configure some interfaces and add some addresses. Once again, I am
going to use overlapping ranges to show that VRF Lite allows for it.

Adding interfaces to a routing instance is configured under the actual interface
configuration hierarchy with the command `ip vrf forward `. If you have an
address already assigned when you run the `ip vrf forwarding ` the address will
be removed. This is done to make sure that conflicts or pollution of the new
routing table doesn't happen unintentionally.


~~~ bash
ios-1(config)#int gi0/0.100
ios-1(config-subif)#ip vrf forwarding Untrust
ios-1(config-subif)#ip address 10.10.10.1 255.255.255.0
ios-1(config-subif)#int gi0/0.200
ios-1(config-subif)#ip vrf forwarding Trust
ios-1(config-subif)#ip address 172.16.10.1 255.255.255.0
ios-1(config-subif)#int gi0/0.300
ios-1(config-subif)#ip vrf forwarding DMZ
ios-1(config-subif)#ip address 10.10.10.1 255.255.255.0
ios-1(config-subif)#int gi0/0.400
ios-1(config-subif)#ip vrf forwarding Trust
ios-1(config-subif)#ip address 192.168.10.1 255.255.255.0

~~~ 

Before we move forward, let's look into some of the show commands around VRFs on IOS.

~~~ bash
ios-1#show ip vrf 
  Name                             Default RD          Interfaces
  DMZ                              <not set>           Gi0/0.300
  Trust                            <not set>           Gi0/0.200
                                                       Gi0/0.400
  Untrust                          <not set>           Gi0/0.100
~~~ 

The command `show ip route` Cisco IOS will not show you anything about the other
routing instances, just the global table.

~~~ bash
ios-1(config-subif)#do show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is 1.1.1.2 to network 0.0.0.0

C    192.168.128.0/24 is directly connected, Dot11Radio0/2/0.1
C    192.168.10.0/24 is directly connected, Dot11Radio0/2/0.10
C    192.168.11.0/24 is directly connected, Dot11Radio0/2/0.3
C    192.168.4.0/24 is directly connected, Dot11Radio0/2/0.4
C    192.168.5.0/24 is directly connected, BVI3
C    1.1.1.0/24 is directly connected, GigabitEthernet0/1
     192.168.255.0/30 is subnetted, 1 subnets
C       192.168.255.0 is directly connected, Tunnel255
     192.168.1.0/32 is subnetted, 1 subnets
C       192.168.1.1 is directly connected, Loopback1
C    192.168.69.0/24 is directly connected, Loopback69
O    192.168.2.0/24 [110/1001] via 192.168.255.1, 1d07h, Tunnel255
S*   0.0.0.0/0 [1/0] via 1.1.1.2
~~~ 

Using the command `show ip route vrf ` we can see into each routing table, or
the use of `show ip route vrf *` will let us see them all at once.

~~~ 
ios-1#show ip route vrf *
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is 1.1.1.1 to network 0.0.0.0

C    192.168.128.0/24 is directly connected, Dot11Radio0/2/0.1
C    192.168.10.0/24 is directly connected, Dot11Radio0/2/0.10
C    192.168.11.0/24 is directly connected, Dot11Radio0/2/0.3
C    192.168.4.0/24 is directly connected, Dot11Radio0/2/0.4
C    192.168.20.0/24 is directly connected, Vlan20
C    192.168.5.0/24 is directly connected, BVI3
C    1.1.1.0/24 is directly connected, GigabitEthernet0/1
     192.168.255.0/30 is subnetted, 1 subnets
C       192.168.255.0 is directly connected, Tunnel255
     192.168.1.0/32 is subnetted, 1 subnets
C       192.168.1.1 is directly connected, Loopback1
C    192.168.69.0/24 is directly connected, Loopback69
O    192.168.2.0/24 [110/1001] via 192.168.255.1, 1d14h, Tunnel255
C    192.168.3.0/24 is directly connected, Vlan3
S*   0.0.0.0/0 [1/0] via 1.1.1.1

Routing Table: Untrust
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/24 is subnetted, 1 subnets
C       10.10.10.0 is directly connected, GigabitEthernet0/0.100

Routing Table: Trust
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.10.0/24 is directly connected, GigabitEthernet0/0.400
     172.16.0.0/24 is subnetted, 1 subnets
C       172.16.10.0 is directly connected, GigabitEthernet0/0.200

Routing Table: DMZ
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/24 is subnetted, 1 subnets
C       10.10.10.0 is directly connected, GigabitEthernet0/0.300
ios-1#
~~~ 
Now lets do a little routing.  Just like in the JunOS example a simple static route should be sufficient.
~~~ bash
ios-1(config)#ip route vrf Trust 0.0.0.0 0.0.0.0 192.168.10.2
~~~ 
The `Trust` routing instance table now looks like the following.
~~~ bash
ios-1(config)#do show ip route vrf Trust

Routing Table: Trust
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is 192.168.10.2 to network 0.0.0.0

C    192.168.10.0/24 is directly connected, GigabitEthernet0/0.400
     172.16.0.0/24 is subnetted, 1 subnets
C       172.16.10.0 is directly connected, GigabitEthernet0/0.200
S*   0.0.0.0/0 [1/0] via 192.168.10.2
~~~ 

### VRF Lite Setup on Juniper ScreenOS {#setup-screenos}
<div class="wp-caption" style="float: right;margin: 5px"><img src="/images/ssg-5-shjpg.jpeg" border="0" alt="SSG-5-SH.jpg.jpeg" width="300" height="60" /></div>

Juniper ScreenOS version 6.2.0r2.0 used here is very new and has been working
very well for me in testing.

There are also a few more limitations on the ScreenOS platform that I need to
make note of. The SSG5 I am using has a limit of only 3 routing instances and
some other limits that you should verify yourself before starting. Using the
command `get license-key` will show all the limits for the hardware. The key
things to look for are: *Vrouters*, *Zones*, and *VLANs*.

~~~ bash
screenos-1-> get license-key 
extended_key        : XXXXXXXXXXXXX+XXXXXXXXXXXXXXXXXXXXXXX+XXXXXXXXXXXX
                      XXXXXXXXXXXXXXXXXXX/
                      XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
                      XXXXXXXXXXXXXXXXXXXXXXXX/
                      XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX+XXXXXXXXXXXX/
                      XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX+XXXXXXXXXXXXXXXX
                      /XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX++XXXXXXXXXXXXXX/
                      XXXXXXXXXXXXXX+XXXXXX+XXXXXXXXXXXXXXXXXXXXXXXXXXXX
                      ==

Sessions:           16064 sessions
Capacity:           unlimited number of users
NSRP:               ActiveActive
VPN tunnels:        40 tunnels
Vsys:               None
Vrouters:           4 virtual routers
Zones:              10 zones
VLANs:              50 vlans
Drp:                Enable
Deep Inspection:    Enable
Deep Inspection Database Expire Date: Disable
Signature pack:     Signature update key is missing
IDP:                Disable
AV:                 Disable(0)
Anti-Spam:          Disable(0)
Url Filtering:      Disable

Update server url: nextwave.netscreen.com/key_retrieval
License key auto update : Disabled
Auto update interval : 0 days
~~~ 

Unlike IOS and JunOS: ScreenOS does not have a concept of Global routing
instance. Every interface must be in routing instances and can not have any
addresses assigned when you move them to a different instance. Due to this, you
really should start off in a different order and create the routing instances
first.

The default ScreenOS puts all interfaces into the `Trust-vr` routing instance so
let's start by checking what is already set up.

~~~ bash
screenos-1-> get vrouter
* indicates default vrouter 
A - AutoExport, R - RIP, N- NHRP, O - OSPF, B - BGP, P - PIM

   ID Name                     Vsys                 Owner     Routes    MRoutes     Flags
    1 untrust-vr               Root                 shared      0/max       0/max       
*   2 trust-vr                 Root                 shared      4/max       0/max       

total 2 vrouters shown and 0 of them defined by user
~~~ 

As you can see there are already 2 routing instances set up. Let's take a look
at the interfaces that belong to each. To do this we need to see what zones are
mapped to which routing instances.

~~~ bash
screenos-1-> get zone  
Total 14 zones created in vsys Root - 8 are policy configurable.
Total policy configurable zones for Root is 8.
;------------------------------------------------------------------------
  ID Name                             Type    Attr    VR          Default-IF   VSYS      
   0 Null                             Null    Shared untrust-vr   wireless0/3  Root                
   1 Untrust                          Sec(L3) Shared trust-vr     ethernet0/0  Root                
   2 Trust                            Sec(L3)        trust-vr     bgroup0      Root                
   3 DMZ                              Sec(L3)        trust-vr     ethernet0/1  Root                
   4 Self                             Func           trust-vr     self         Root                
   5 MGT                              Func           trust-vr     null         Root                
   6 HA                               Func           trust-vr     null         Root                
  10 Global                           Sec(L3)        trust-vr     null         Root                
  11 V1-Untrust                       Sec(L2) Shared trust-vr     v1-untrust   Root                
  12 V1-Trust                         Sec(L2) Shared trust-vr     v1-trust     Root                
  13 V1-DMZ                           Sec(L2) Shared trust-vr     v1-dmz       Root                
  14 VLAN                             Func    Shared trust-vr     vlan1        Root                
  15 V1-Null                          Sec(L2) Shared trust-vr     l2v          Root                
  16 Untrust-Tun                      Tun            trust-vr     hidden.1     Root                
;------------------------------------------------------------------------
~~~ 

Now we have to map the interfaces to the zones. (Yes, it may seem a little
convoluted but it does make sense for a firewall platform).

~~~ bash
screenos-1-> get interface 

A - Active, I - Inactive, U - Up, D - Down, R - Ready 

Interfaces in vsys Root: 
Name           IP Address                        Zone        MAC            VLAN State VSD      
serial0/0      0.0.0.0/0                         Null        N/A               -   D   -  
eth0/0         0.0.0.0/0                         Untrust     0017.cb80.9f40    -   U   -  
eth0/1         0.0.0.0/0                         DMZ         0017.cb80.9f45    -   D   -  
wireless0/0    192.168.2.1/24                    Trust       0017.cb80.9f55    -   D   -  
wireless0/1    0.0.0.0/0                         Null        0017.cb80.9f56    -   D   -  
wireless0/2    0.0.0.0/0                         Null        0017.cb80.9f57    -   D   -  
wireless0/3    0.0.0.0/0                         Null        0017.cb80.9f58    -   D   -  
bgroup0        192.168.1.1/24                    Trust       0017.cb80.9f4b    -   U   -  
  eth0/2       N/A                               N/A         N/A               -   U   -
  eth0/3       N/A                               N/A         N/A               -   D   -
  eth0/4       N/A                               N/A         N/A               -   D   -
  eth0/5       N/A                               N/A         N/A               -   D   -
  eth0/6       N/A                               N/A         N/A               -   D   -
bgroup1        0.0.0.0/0                         Null        0017.cb80.9f4c    -   D   -  
bgroup2        0.0.0.0/0                         Null        0017.cb80.9f4d    -   D   -  
bgroup3        0.0.0.0/0                         Null        0017.cb80.9f4e    -   D   -  
vlan1          0.0.0.0/0                         VLAN        0017.cb80.9f4f    1   D   -  
null           0.0.0.0/0                         Null        N/A               -   U   0  
~~~ 

We now have all the information we need to begin the process. Here is a
simplified table to make moving forward a little easier:

##### Current
| Interface   | Zone     | Routing Instance  |
|-------------|----------|-------------------|
| serial0/0   | Null     | trust-vr          |
| eth0/0      | Untrust  | trust-vr          |
| eth0/1      | DMZ      | trust-vr          |
| wireless0/0 | Trust    | trust-vr          |

Now let's start by creating the one routing instance that is not already setup by default.

~~~ bash
screenos-1-> set vrouter name dmz-vr
~~~ 
Now let's see how this shows up on the device.

~~~ bash
creenos-1-> get vrouter
* indicates default vrouter 
A - AutoExport, R - RIP, N- NHRP, O - OSPF, B - BGP, P - PIM

   ID Name                     Vsys                 Owner     Routes    MRoutes     Flags
    1 untrust-vr               Root                 shared      0/max       0/max       
*   2 trust-vr                 Root                 shared      4/max       0/max       
 1025 dmz-vr                   Root                 user        0/max       0/max       

total 3 vrouters shown and 1 of them defined by user
~~~ 

Due to the limitations of not allowing the movement of a zone between routing
instances when there are interfaces within them, we need to move things around
first. Let's start by moving all the interfaces that are in the *Trust* and
*DMZ* zones to a holder zone named *Null*.

~~~ bash
screenos-1-> set interface eth0/0 zone Null
screenos-1-> set interface eth0/1 zone Null
~~~ 

Now we need to move the zones to the correct routing instances, and while we're
at it let's move the interfaces back and create new sub-interfaces.

~~~ bash
screenos-1-> set zone Untrust vrouter untrust-vr
screenos-1-> set zone DMZ vrouter dmz-vr
screenos-1-> set interface eth0/0 zone Untrust
screenos-1-> set interface eth0/1 zone DMZ
screenos-1-> set interface eth0/0.1 tag 100 zone Untrust
screenos-1-> set interface eth0/0.2 tag 200 zone Trust
screenos-1-> set interface eth0/0.3 tag 300 zone DMZ
screenos-1-> set interface eth0/0.4 tag 400 zone Trust

~~~ 

Finally, let's setup the interface addresses.

~~~ bash
screenos-1-> set interface eth0/0.1 ip 10.10.10.1/24
screenos-1-> set interface eth0/0.2 ip 172.16.10.1/24
screenos-1-> set interface eth0/0.3 ip 10.10.10.1/24
screenos-1-> set interface eth0/0.4 ip 192.168.10.1/24
~~~ 

Now we should take a look and see that everything has come out the way we
expected. First, the interfaces:

~~~ bash
screenos-1-> get interface 

A - Active, I - Inactive, U - Up, D - Down, R - Ready 

Interfaces in vsys Root: 
Name           IP Address                        Zone        MAC            VLAN State VSD      
serial0/0      0.0.0.0/0                         Null        N/A               -   D   -  
eth0/0         0.0.0.0/0                         Untrust     0017.cb80.9f40    -   U   -  
eth0/0.1       0.0.0.0/0                         Untrust     0017.cb80.9f40  100   U   -  
eth0/0.2       0.0.0.0/0                         Trust       0017.cb80.9f40  200   U   -  
eth0/0.3       0.0.0.0/0                         DMZ         0017.cb80.9f40  300   U   -  
eth0/0.4       0.0.0.0/0                         Trust       0017.cb80.9f40  400   U   -  
eth0/1         0.0.0.0/0                         DMZ         0017.cb80.9f45    -   D   -  
wireless0/0    192.168.2.1/24                    Trust       0017.cb80.9f55    -   D   -  
wireless0/1    0.0.0.0/0                         Null        0017.cb80.9f56    -   D   -  
wireless0/2    0.0.0.0/0                         Null        0017.cb80.9f57    -   D   -  
wireless0/3    0.0.0.0/0                         Null        0017.cb80.9f58    -   D   -  
bgroup0        192.168.1.1/24                    Trust       0017.cb80.9f4b    -   U   -  
  eth0/2       N/A                               N/A         N/A               -   U   -
  eth0/3       N/A                               N/A         N/A               -   D   -
  eth0/4       N/A                               N/A         N/A               -   D   -
  eth0/5       N/A                               N/A         N/A               -   D   -
  eth0/6       N/A                               N/A         N/A               -   D   -
bgroup1        0.0.0.0/0                         Null        0017.cb80.9f4c    -   D   -  
bgroup2        0.0.0.0/0                         Null        0017.cb80.9f4d    -   D   -  
bgroup3        0.0.0.0/0                         Null        0017.cb80.9f4e    -   D   -  
vlan1          0.0.0.0/0                         VLAN        0017.cb80.9f4f    1   D   -  
null           0.0.0.0/0                         Null        N/A               -   U   0  
~~~ 

Now the routing instances:

~~~ bash
screenos-1-> get route
H: Host C: Connected S: Static A: Auto-Exported
I: Imported R: RIP P: Permanent D: Auto-Discovered
N: NHRP
iB: IBGP eB: EBGP O: OSPF E1: OSPF external type 1
E2: OSPF external type 2 trailing B: backup route


IPv4 Dest-Routes for <untrust-vr> (2 entries)
;--------------------------------------------------------------------------------------
         ID          IP-Prefix      Interface         Gateway   P Pref    Mtr     Vsys
;--------------------------------------------------------------------------------------
*         2      10.10.10.1/32       eth0/0.1         0.0.0.0   H    0      0     Root
*         1      10.10.10.0/24       eth0/0.1         0.0.0.0   C    0      0     Root



IPv4 Dest-Routes for <trust-vr> (8 entries)
;--------------------------------------------------------------------------------------
         ID          IP-Prefix      Interface         Gateway   P Pref    Mtr     Vsys
;--------------------------------------------------------------------------------------
*         5     172.16.10.0/24       eth0/0.2         0.0.0.0   C    0      0     Root
*         8    192.168.10.1/32       eth0/0.4         0.0.0.0   H    0      0     Root
*         4     192.168.1.1/32        bgroup0         0.0.0.0   H    0      0     Root
          2     192.168.2.1/32    wireless0/0         0.0.0.0   H    0      0     Root
          1     192.168.2.0/24    wireless0/0         0.0.0.0   C    0      0     Root
*         3     192.168.1.0/24        bgroup0         0.0.0.0   C    0      0     Root
*         7    192.168.10.0/24       eth0/0.4         0.0.0.0   C    0      0     Root
*         6     172.16.10.1/32       eth0/0.2         0.0.0.0   H    0      0     Root



IPv4 Dest-Routes for <dmz-vr> (2 entries)
;--------------------------------------------------------------------------------------
         ID          IP-Prefix      Interface         Gateway   P Pref    Mtr
;--------------------------------------------------------------------------------------
*         2      10.10.10.1/32       eth0/0.3         0.0.0.0   H    0      0         
*         1      10.10.10.0/24       eth0/0.3         0.0.0.0   C    0      0         


~~~ 

> Based on the findings of the report, my conclusion was that this idea was not
> a practical deterrent for reasons which at this moment must be all too obvious.
>
> Dr. Strangelove

## Footnotes {#footnotes}

[^1]: MAN is a Metropolitan Area Network: [Wikipedia](http://en.wikipedia.org/wiki/Metropolitan_Area_Network)

[^2]: Yes, yes. I know I could do everything at once and commit last, and that is one of the reasons I love JunOS, but this is also about building and seeing each change and how it affects the overall router

[^3]:Table of Vender and VRF naming conventions
   Vendor  | OS       | VRF-Lite       | VRF   | Notes
   ---------|----------|----------------|-------|------
   Juniper | JunOS    | Virtual Router | VRF   | JunOS has many others ways of preforming VRF functions. More details <a href="http://www.juniper.net/techpubs/software/junos/junos85/swconfig85-vpns/frameset.html">here</a>
   Juniper | ScreenOS | Virtual Router | _N/A_ |
   Cisco   | IOS      | VRF Lite       | VRF   |
   Cisco   | NX-OS    | VRF Lite       | VRF   |
   Cisco   | ASA      | Contexts       | _N/A_ |
   Cisco   | PIXOS    | _N/A_          | _N/A_ |

[^4]: I should take a second and also point out that Cisco has a long and _s.l.o.w_ history of making managements services available via a vrf.  In fact, so many features cannot be enabled inside a VRF that most just use the global routing table for management and push all production traffic into VRFs.

