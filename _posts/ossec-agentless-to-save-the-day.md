---
layout: post
title: "Ossec: Agentless to save the day"
date: 2009-10-02 
categories: 
  - ossec
tags: 
  - security
  - ossec
  - agentless
  - unix 
series:
  - "Praetorian Prefect"
slug: ossec-agentless-to-save-the-day
aliases:
  - /ossec-agentless-to-save-the-day.html
author: Jeremy Rossi
discription: |
    OSSEC is a Host Intrusion Detection System (HIDS) in name, but in 
    reality it is far more.  It's able to look for rootkits, monitor logs 
    (LIDS), and even actively respond to defined events.  While all 
    these features are great, the unsung hero is agentless monitoring.
---



>  Lois, Clark Kent may seem like just a mild-mannered reporter, but
> listen, not only does he know how to treat his editor-in-chief with the
> proper respect, not only does he have a snappy, punchy prose style, but
> he is, in my forty years in this business, the fastest typist I've ever
> seen.
>
> `Perry White`

Michael Starks from [Immutable Security](http://www.immutablesecurity.com/) published the "Week of
OSSEC" all last week (find their links at the end of article), and it
was a great setup of posts.

With all the hard work done by Michael in his "Week of OSSEC", I figured
I should follow up with a few posts of my own about this great tool.
I am **NOT** going to do a week of posts, but will try to get as much
information out as I can.

### OSSEC

<img src="http://praetorianprefect.com/wp-content/uploads/2009/11/Screen-shot-2009-11-02-at-8.06.14-PM.png" border="0" alt="Screen shot 2009-11-02 at 8.06.14 PM.png" width="66" height="64" />

OSSEC is a Host Intrusion Detection System (HIDS) in name, but in
reality it is far more. It's able to look for rootkits, monitor logs
(LIDS), and even actively respond to defined events. While all these
features are great, the unsung hero is agentless monitoring.

Agentless security monitoring is really a great feature that does not
get explored often enough, so I am going to show how to get it up and
running and then get it monitoring remote hosts.

### Installing OSSEC {#ossec-install}

This is going to be one of the fastest OSSEC install
instructions on the internet. For full details the main 
[OSSEC website](http://www.ossec.net/main/documentation/) which covers this
topic with more detail. Key things to note here is that I have installed
it as a server. I could have installed OSSEC locally and we would have
still been able to do whatever was needed.

My install log for OSSEC 2.2 is <a title="install-ossec-v2.2.txt" href="http://praetorianprefect.com/wp-content/uploads/2009/10/install-ossec-v2.2.txt">here</a>.

### Enabling agentless {#agentless-enable}

To make use of agentless security monitoring, it first
needs to be enabled. Full details also on the 
[OSSEC webpage](http://www.ossec.net/main/manual/manual-agentless-monitoring/).

#### Agentless Requirements

For most of the built-in agentless monitoring scripts, `expect` is
needed to function. In this example on OpenBSD 4.5, adding the `expect`
package is simple with `pkg_add`.

```bash
obsd46# pkg_add http://openbsd.mirror.frontiernet.net/pub/OpenBSD/4.5/packages/i386/expect-5.43.0p0-no_tk.tgz
tcl-8.4.19: complete
expect-5.43.0p0-no_tk: complete
--- tcl-8.4.19 -------------------
You may wish to add /usr/local/lib/tcl8.4/man to /etc/man.conf
```

#### Turning on Agentless

Now we need to enable agentless by running the following command:

```bash
obsd46# /var/ossec/bin/ossec-control enable agentless
```

#### Adding a host.

We need to add a host to agentlessly monitor. If we were to authenticate
using a password for host `172.17.20.20` we would use the following:

```bash
obsd46# /var/ossec/agentless/register_host.sh add agentless@172.17.20.20
```

While using a password does work, the preferred method would be to use
SSH keys to provide the access level needed. To setup that method of
access, you first need to create ssh keys for the user `ossec` which is
the account the agentless scripts runs as.

```bash
obsd46# sudo -u ossec ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/var/ossec/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/ossec/.ssh/id_rsa.
Your public key has been saved in /var/ossec/.ssh/id_rsa.pub.
The key fingerprint is:
b8:c3:47:9a:33:09:5c:eb:54:a0:82:39:a6:06:63:08 ossec@obsd45.ptnsecurity.com
The key's randomart image is:
+--[ RSA 2048]----+
|E     .          |
|oo   . .         |
|Bo. . . .        |
|=o o . +         |
|..  o + S        |
|.    = *         |
|      @ .        |
|       =         |
|                 |
+-----------------+
```

Now that the SSH keys are present, we can add the host without a
password. The special command line argument used with `register_host.sh`
is `NOPASS` in all capitals, which will tell OSSEC supplied scripts to
make use of SSH keys.

```
obsd46# /var/ossec/agentless/register_host.sh add root@172.17.20.20 NOPASS
```
#### Enabling SSH key on the host to be monitored.

You will now need to securely get the contents of
`/var/ossec/.ssh/id_rsa.pub` to 172.17.20.20.

Using SSH and the password for a single time will make this simple. This
will create the `/root/.ssh` if it is not already created, but might
throw an error as it does if the directory is already present. This is
not a problem and can be ignored.


```bash
obsd46# cat /var/ossec/.ssh/id_rsa.pub | ssh root@172.17.20.20 "( mkdir /root/.ssh/;  cat - >> /root/.ssh/authorized_keys )"
root@172.17.20.20's password:
mkdir: cannot create directory `/root/.ssh/': File exists
obsd46# ssh root@172.17.20.20 "cat  /root/.ssh/authorized_keys "
root@172.17.20.20's password:
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAzyTBo7CqkI0TISR9S+KPS/gYY60nkD7Qe8wTTXrAEFvPNFJ
NJJpVVKsij6zw86lvTZ6hx9ib1M+MXvt+70uF/z1hYwnYrczR2TR03Z5nwOUA9OK61nBWXVwCi9GsQs6Oeo
mY9vkBDoKzB52+TKKSk9ZoC+HYPiT5SaiHZvMOV7kWuwF67lnYwlG5FdkRdOiXp7DcRjje4/Hixg7RLLl7o
dEXpIakzGfalt3yQDmwvSUZhyg3OuoKimTeNiKU/jlHlmEPuDZpiQe6QhFH38EeEIZTdHsYITodl8sY+n9I
eNMalGIHPs+bph+qcK+6cOb1RGaeGqJBFjaqPUyismz0bw== ossec@obsd45.ptnsecurity.com
```

We can also verify that it worked with the following command.

```bash
obsd46# sudo -u ossec ssh root@172.17.20.20
The authenticity of host '172.17.20.20 (172.17.20.20)' can't be established.
RSA key fingerprint is 14:cd:f2:e9:c3:5b:07:28:68:75:a7:b5:88:c2:6b:77.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.17.20.20' (RSA) to the list of known hosts.
Last login: Tue Oct  6 12:40:05 2009 from 172.17.20.154
[linux26.ptnsecurity.com ~]# exit
```

#### Add the agentless host to ossec.conf

While we have setup and prepared everything to allow agentless
security monitoring of `172.17.20.20` we have not told ossec to
make use of it. To simplify adding agentless to the config, we
are going to make use of the python library and tools I created
[ossec-hids-tools](http://bitbucket.org/jrossi/ossec-hids-tools/).

First, let's check to see what agentless hosts have been configured, and
just like a good unix program, it should not output anything if nothing
happens.

```bash
obsd46# ossec-config --section agentless --show
```

Next, add our host to the configuration. I am using the OSSEC supplied
script `ssh_integrity_check_linux`. This script will login to the remote
host and send back to the OSSEC server via stdout an MD5 and SHA1 hash
of every single file inside the paths specified in the arguments. To
demonstrate the output from the server, let's test the script and review
said output.

All testing of agentless scripts must be run from the directory
`/var/ossec/` unless you compiled a different install location.

```bash
obsd46# cd /var/ossec
obsd46# sudo -u ossec ./agentless/ssh_integrity_check_linux root@172.17.20.20 /etc
spawn ssh root@172.17.20.20
Last login: Mon Nov  2 17:53:23 2009 from 172.17.20.131
[tss-uvc-01v.ptn.local ~]#
INFO: Started.
t -d " " -f 1` && echo FWD: `stat --printf "%s:%a:%u:%g" $i`:$md5:$sha1 $i; done; exit md5=`md5sum $i | cut -d " " -f 1` && sha1=`sha1sum $i | cu
INFO: Starting.
FWD: 14612:644:0:0:509377d820692110c7a6cc83ef2c2da8:bf610c1fa14d84d8b3b44ec80b81788457f77420 /etc/sound/events/gtk-events-2.soundlist
FWD: 22291:644:0:0:d6139aa9554d4997ea25ec2d56095f51:26b9ae7784943eecaeb2dcd4b2ae3a32371d61c8 /etc/sound/events/gnome-2.soundlist
FWD: 83:644:0:0:9f87609f65b51761657c7d67881ae582:de82c03c535e9deb16aed94153883280891da2d7 /etc/modprobe.d/blacklist-firewire
^C^C#
```

I only let the script run for a few seconds to see the output, but the
key things to notice are the lines beginning with "INFO" or "FWD".

Anything that starts with "`INFO`" is logged to the
`/var/ossec/logs/ossec.log` file for debugging and troubleshooting, we
will make use of this later on in this blog. The "`FWD`" tag at the
beginning of the line lets the OSSEC server store the HASH information.
Where this becomes useful is when a file's contents change, the HASH
will in turn change and OSSEC is able to notify you when this happens.

Now let's complete adding our host to the OSSEC configuration.

```bash
obsd46# ossec-config --section agentless --add --host root@172.17.20.20 --type ssh_integrity_check_linux \
--state periodic --argv "/bin /etc /sbin"
```

Let's verify it's what we expect.

```bash
obsd46# ossec-config --section agentless --show
----------------------------------------
type: ssh_integrity_check_linux
frequency: 3600
host: root@172.17.20.20
state: periodic
arguments: /bin /etc /sbin
```
Time to restart the deamons for the changes to take effect.
```
obsd46# /var/ossec/bin/ossec-control stop
Killing ossec-monitord ..
Killing ossec-logcollector ..
ossec-remoted not running ..
Killing ossec-syscheckd ..
Killing ossec-analysisd ..
Killing ossec-maild ..
ossec-execd not running ..
ossec-agentlessd not running ..
OSSEC HIDS v2.2 Stopped
obsd46# /var/ossec/bin/ossec-control start
Starting OSSEC HIDS v2.2 (by Trend Micro Inc.)...
Started ossec-agentlessd...
Started ossec-maild...
Started ossec-execd...
Started ossec-analysisd...
Started ossec-logcollector...
Started ossec-remoted...
Started ossec-syscheckd...
Started ossec-monitord...
Completed.
```

### Testing agentless {#agentless-test}

Checking the log files, we can see what the agentless security monitor
has done so far.

```bash
obsd46# grep agentless logs/ossec.log
2009/09/21 14:59:49 ossec-agentlessd: INFO: Started (pid: 15320).
2009/09/21 14:59:51 ossec-agentlessd: INFO: Test passed for 'ssh_integrity_check_linux'.
2009/09/21 15:00:53 ossec-agentlessd: INFO: ssh_integrity_check_linux: root@172.17.20.20: Started.
2009/09/21 15:00:53 ossec-agentlessd: INFO: ssh_integrity_check_linux: root@172.17.20.20: Starting.
2009/09/21 15:01:34 ossec-agentlessd: INFO: ssh_integrity_check_linux: root@172.17.20.20: Finished.
```

Now we have one last thing to do to see that it's working as expected,
make a change to the file system on `172.17.20.20` that the ossec will
notice on the next run. I am going to change the root password for now.

```bash
obsd46# ssh -i /var/ossec/.ssh/id_rsa root@172.17.20.20
Last login: Tue Oct  6 14:38:48 2009 from 172.17.20.154
[linux26.ptnsecurity.com ~]# passwd
Changing password for user root.
New UNIX password:
Retype new UNIX password:
passwd: all authentication tokens updated successfully.
[linux26.ptnsecurity.com ~]# exit
```

Take a look at the logs for ossec-agentlessd to check the host. Again,
we see that it completed another scan.

```bash
obsd46# grep agentless logs/ossec.log
2009/09/21 15:18:27 ossec-agentlessd: INFO: ssh_integrity_check_linux: root@172.17.20.20: Started.
2009/09/21 15:18:27 ossec-agentlessd: INFO: ssh_integrity_check_linux: root@172.17.20.20: Starting.
2009/09/21 15:18:46 ossec-syscheckd: INFO: Finished creating syscheck database (pre-scan completed).
2009/09/21 15:19:06 ossec-agentlessd: INFO: ssh_integrity_check_linux: root@172.17.20.20: Finished.
```

Note that we also received the following email notifying that the
password has changed, a message that is very useful to report.

```bash
OSSEC HIDS Notification.
2009 Sep 21 15:19:00

Received From: (ssh_integrity_check_linux) root@172.17.20.20->syscheck
Rule: 550 fired (level 7) -> "Integrity checksum changed."
Portion of the log(s):

Integrity checksum changed for: '/etc/shadow'
Old md5sum was: '0d92e12c92f3edcf9d8876ea57c5f677'
New md5sum is : '2bd51b61dea17c5682fb2c0cf4f92c63'
Old sha1sum was: '2270c03a920ef8dd50e11cefdef046a8660f7a29'
New sha1sum is : 'd9518ea9022b10d07f81925c6d7f2abb4364b548'

--END OF NOTIFICATION
```

---
Week of OSSEC Links:

* Day 1: [Detecting World-Writable Files](http://www.immutablesecurity.com/index.php/2009/10/25/week-of-ossec-day-1-detecting-world-writable-files/)
* Day 2: [Detecting New Files](http://www.immutablesecurity.com/index.php/2009/10/26/week-of-ossec-day-2-detecting-new-files/)
* Day 3: [Using Variables](http://www.immutablesecurity.com/index.php/2009/10/27/week-of-ossec-day-3-use-variables/)
* Day 4: [Using Groups](http://www.immutablesecurity.com/index.php/2009/10/28/week-of-ossec-day-4-using-groups/)
* Day 5: [Reusing Rule IDs](http://www.immutablesecurity.com/index.php/2009/10/29/week-of-ossec-day-5-reusing-rule-ids/)
* Day 6: [Developing a Tuning Strategy](http://www.immutablesecurity.com/index.php/2009/10/30/week-of-ossec-day-6-developing-a-tuning-strategy/)
* Day 7: [Developing a Workflow](http://www.immutablesecurity.com/index.php/2009/10/31/week-of-ossec-day-7-developing-a-workflow/)


