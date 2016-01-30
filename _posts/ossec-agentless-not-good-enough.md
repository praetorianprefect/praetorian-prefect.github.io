---
layout: post
title: "Ossec: Agentless....it's good, but not good enough. "
date: 2009-10-05
categories:
  - ossec
  - programming
tags: 
  - security
  - ossec
  - agentless
  - unix 
  - c
  - programming
series:
  - "Praetorian Prefect"
series:
  - "Praetorian Prefect"
slug: ossec-agentless-good-but-not-good-enough
author: Jeremy Rossi
aliases:
  - /ossec-agentless-good-but-not-good-enough.html
summary: 
    In working with OSSEC agentless for some time now I have come across
    some limitations in the implementation that I felt needed to be
    addressed. As OSSEC agentless is designed to preform `syscheck`
    functions on remote hosts, more general features are hard (if not
    impossible) to write into a script.
---


<div class="wp-caption" style="float: right;margin: 5px;margin-left: 42px;margin-right: 21px;"><img src="http://praetorianprefect.com/wp-content/uploads/2009/11/Screen-shot-2009-11-02-at-8.06.14-PM.png" border="1" alt="ossec_logo" width="66" height="64" /></div>

In working with OSSEC agentless for some time now I have come across
some limitations in the implementation that I felt needed to be
addressed. As OSSEC agentless is designed to preform `syscheck`
functions on remote hosts, more general features are hard (if not
impossible) to write into a script.

Currently in OSSEC, agentless scripts are limited to the following commands: 

Command | Description 
--------|------------------------------------------------------------------------------------------------------------------------------------------------------
`INFO:` | The string following INFO will be logged to `/var/ossec/logs/ossec.log` by OSSEC for debugging.  
`ERROR:`| Error needs to be reported.  The string following this command is forwarded to the OSSEC manager, and the OSSEC process closes down the script. 
`STORE:`| All the lines that follow this command will be added, stored, and compared to previous runs of the script. 
`FWD:`  | The string following FWD is a colon delimited list of stats on a given file.  Example: `FWD: <size>:<permissions>:<uid>:<gid>:<md5>:<sha1> <path & file>`  

Given the choices listed here more advanced agentless scripts are just
not reasonably possible. I require the ability to pass more information
to the OSSEC agentless process and have it raise alerts based on this
information.

### Solution patch OSSEC

So I starting digging into the OSSEC code. I am not a C coder, I don't
even play one on TV, but the OSSEC's code is clear and has just enough
comments to allow me to understand how things function. Once I saw where
the communication happens between ossec-agentless and it's subprocess I
was quickly able to add a new OSSEC Agentless Command.

Command | Description 
--------|--------------------------------------------------------------------------------------------------------------------------------
`LOG:`  | The string following LOG: will be passed into `ossec-analysisd` and processed like all other log messages. 

This simple command allow scripts to generate messages that will get
processed by the standard OSSEC decoders and rules.

* Direct download of patch: <a href="http://praetorianprefect.com/wp-content/uploads/2009/11/agentless.patch.txt" title="agentless.patch.txt">agentless.patch.txt</a>

### Patching OSSEC

The patch I created works with the current code release of OSSEC. To
apply the patch, first download OSSEC version 2.2 from the website. In
the instructions below, I have changed to the tmp directory first as we
will be removing the source files once we have finished the install.

```bash
obsd46# cd /tmp 
obsd46# ftp http://www.ossec.net/files/ossec-hids-2.2.tar.gz
Trying 75.126.165.213...
Requesting http://www.ossec.net/files/ossec-hids-2.2.tar.gz
100% |******************************************************************|   692 KB    00:03    
Successfully retrieved file.
```

Now expand the downloaded archive and change into the newly created
directory `ossec-hids-2.2`.

```bash
obsd46# tar xfz ossec-hids-2.2.tar.gz                                                                                                                                                   
obsd46# cd ossec-hids-2.2       
```

This is where most of the work will happen, but first we need to download the patch. 

```bash
obsd46# ftp http://praetorianprefect.com/wp-content/uploads/2009/11/agentless.patch.txt                             
Trying 75.101.150.229...
Requesting http://praetorianprefect.com/wp-content/uploads/2009/11/agentless.patch.txt
100% |******************************************************************| 10278       00:00    
Successfully retrieved file.
```

Now we just apply the patch. We will use the `patch` command do
this, but using the argument `-p1` to apply the patch cleanly to all
sub-directories.

```bash
obsd46# patch -p1 < agentless.patch.txt  
Hmm... this looks like a unified diff to me...
The text leading up to this was:
 |-------------------------
 |diff -r 55072a52aaa4 -r 673c04be67e9 etc/decoder.xml
 |--- a/etc/decoder.xml  Wed Nov 04 20:51:36 2009 -0500
 |+++ b/etc/decoder.xml  Fri Nov 06 19:53:36 2009 +0000
 |-------------------------
Patching file etc/decoder.xml using Plan A...
Hunk #1 succeeded at 70.
Hunk #2 succeeded at 1498.
Hmm...  The next patch looks like a unified diff to me...
The text leading up to this was:
 |-------------------------
 |diff -r 55072a52aaa4 -r 673c04be67e9 etc/rules/agentless_rules.xml
 |--- /dev/null  Thu Jan 01 00:00:00 1970 +0000
 |+++ b/etc/rules/agentless_rules.xml    Fri Nov 06 19:53:36 2009 +000 0
 |-------------------------
(Creating file etc/rules/agentless_rules.xml...)
Patching file etc/rules/agentless_rules.xml using Plan A...
Empty context always matches.
Hunk #1 succeeded at 1.
Hmm...  The next patch looks like a unified diff to me...
The text leading up to this was:
 |-------------------------
 |diff -r 55072a52aaa4 -r 673c04be67e9 etc/rules/ossec_rules.xml
 |--- a/etc/rules/ossec_rules.xml        Wed Nov 04 20:51:36 2009 -0500
 |+++ b/etc/rules/ossec_rules.xml        Fri Nov 06 19:53:36 2009 +0000
 |-------------------------
Patching file etc/rules/ossec_rules.xml using Plan A...
Hunk #1 succeeded at 153.
Hmm...  The next patch looks like a unified diff to me...
The text leading up to this was:
 |-------------------------
 |diff -r 55072a52aaa4 -r 673c04be67e9 etc/templates/config/rules.template
 |--- a/etc/templates/config/rules.template      Wed Nov 04 20:51:36 2009 -0500
 |+++ b/etc/templates/config/rules.template      Fri Nov 06 19:53:36 2009 +0000
 |-------------------------
Patching file etc/templates/config/rules.template using Plan A...
Hunk #1 succeeded at 44.
Hmm...  The next patch looks like a unified diff to me...
The text leading up to this was:
 |-------------------------
 |diff -r 55072a52aaa4 -r 673c04be67e9 src/agentlessd/scripts/nmap_policy
 |--- /dev/null  Thu Jan 01 00:00:00 1970 +0000
 |+++ b/src/agentlessd/scripts/nmap_policy       Fri Nov 06 19:53:36 2009 +0000
 |-------------------------
(Creating file src/agentlessd/scripts/nmap_policy...)
Patching file src/agentlessd/scripts/nmap_policy using Plan A...
Empty context always matches.
Hunk #1 succeeded at 1.
done 
```

Now we have a completed all the OSSEC 2.2 code patches for the
expanded agentless features. At this point you will need to
compile and install OSSEC. For full details the main 
[OSSEC website](http://www.ossec.net/main/documentation/) covers this topic in
more detail. A key thing to note here is that OSSEC has to be installed
as a server or locally.

Please see my article on how to enable [OSSEC agentless monitoring](http://praetorianprefect.com/archives/2009/11/ossec-agentless-to-save-the-day/).

### Making use of the new features

Now that we have a patched and installed version of OSSEC we can take
advantage of the newly added features. Included with the patch is a
new Agentless OSSEC script `nmap_policy`. This script is really not
designed for production use, rather it's geared to show how to use the
new agentless features.

Let's get into the details. Start by running the new script and looking
at the output. I should note that this script uses `python` and needs at
least version 2.5 in order to parse the xml output from `nmap`.

```bash
obsd45# (cd /var/ossec && ./agentless -b 21,23,80 -n 172.17.20.20/32 )
INFO: Starting
INFO: running `nmap -p 21,23,80 -oX - 172.17.20.0/24` command
INFO: completed `nmap -p 21,23,80 -oX - 172.17.20.0/24` command
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.1 (00:0E:83:A9:E6:80 Cisco Systems)
LOG:alert=11 Policy violation port 23 (telnet) is open on host 172.17.20.1 (00:0E:83:A9:E6:80 Cisco Systems)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.19 (00:18:8B:1E:27:A5 Dell)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.20 (00:0C:29:84:72:11 VMware)
LOG:alert=11 Policy violation port 21 (ftp) is open on host 172.17.20.20 (00:0C:29:84:72:11 VMware)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.21 (00:0C:29:8D:39:E4 VMware)
LOG:alert=11 Policy violation port 21 (ftp) is open on host 172.17.20.21 (00:0C:29:8D:39:E4 VMware)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.31 (00:0C:29:29:CF:35 VMware)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.32 (00:0C:29:58:5F:C1 VMware)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.57 (00:1E:0B:9D:C0:03 Hewlett Packard)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.91 (00:14:38:D8:01:DD Hewlett Packard)
LOG:alert=11 Policy violation port 21 (ftp) is open on host 172.17.20.134 (00:1E:C2:03:2D:E8 Apple)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.202 (00:19:B9:24:7E:F2 Dell)
LOG:alert=11 Policy violation port 80 (http) is open on host 172.17.20.203 (00:19:B9:24:7E:F2 Dell)
INFO: Ending
```

So what this script does is run `nmap` and looks for ports that are open
and not allowed per an internal policy. In this example I checked for
http, telnet, and ftp, but the selection of ports is configurable with
the `-b`/`--badport` arguments. The second argument `-n`/`--network` is
used to specify which IP addresses to scan. The format of this option is
very liberal, in fact any valid `nmap` network specification will work.

Just as I specified above any string following the `LOG:` OSSEC
agentless command will be pushed to the `ossec-analysisd` process for
decoding and rules filtering.

As part of the patch I have also included an updated `decode.xml` and
a new `agentless_rules.xml` to begin the first level of processing
of output from the scripts. Using `ossec-logtest` we can see this in
action, but due to how `ossec-agentlessd` processes the messages we need
to slightly modify the output for it to work with `ossec-logtest`.

```bash
obsd46# (cd /var/ossec && ./bin/ossec-logtest )                                                                                                                                                            
2009/11/06 20:48:28 ossec-testrule: INFO: Started (pid: 9789).
ossec-testrule: Type one log per line.

Agentless: Log:alert=11 Policy violation port 80 (http) is open on host 172.17.20.1 (00:0E:83:A9:E6:80 Cisco Systems)


**Phase 1: Completed pre-decoding.
       full event: 'Agentless: Log:alert=11 Policy violation port 80 (http) is open on host 172.17.20.1 (00:0E:83:A9:E6:80 Cisco Systems)'
       hostname: 'a'
       program_name: '(null)'
       log: 'Agentless: Log:alert=11 Policy violation port 80 (http) is open on host 172.17.20.1 (00:0E:83:A9:E6:80 Cisco Systems)'

**Phase 2: Completed decoding.
       decoder: 'agentless'
       id: '11'
       extra_data: 'Policy violation port 80 (http) is open on host 172.17.20.1 (00:0E:83:A9:E6:80 Cisco Systems)'

**Phase 3: Completed filtering (rules).
       Rule id: '10011'
       Level: '11'
       Description: 'Agentless.'
**Alert to be generated.
```

You can see from the output that a level 11 alert would be generated
for the line we just tested with `ossec-logtest`. In the case of the
full output of the `nmap_policy` script it has 13 `LOG:` lines returned
and would have generated 13 alerts. Needless to say this is a lot of
alerts, so it's up to you to tune and configure this correctly for your
environment.

In our lab here at [Praetorian](http://www.praetoriansecuritygroup.com)
we don't ever want to see the telnet port open. So lets make this script
live, but only checking for telnet. I am going to once again make use of
[ossec-hids-tools](http://bitbucket.org/jrossi/ossec-hids-tools/) to add
the new agentless monitoring. As is always the case a restart of OSSEC
will be needed.

```bash
obsd46# ossec-config --section agentless --add --host jrossi@172.17.20.0 --type nmap_policy 
--frequency 86400 --state periodic --argv "-p 23 -n 172.17.20.0/24"
obsd46# (cd /var/ossec && ./bin/ossec-control restart )
```

While adding this new agentless script I had to specify a `--host`
argument. This is required for OSSEC agentless as the host field is
**NOT** optional. In the case of the script `nmap_policy` it will have
no effect, but this needs to be taken into account when writing your own
scripts as the first argument passed will always be what you specified
as the host.

