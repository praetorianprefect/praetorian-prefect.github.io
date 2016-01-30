---
layout: post
categories: 
  - programming
  - community
date: 2016-01-02T18:42:35-05:00
description: ""
tags: 
  - ossec 
  - make
  - makefile
  - c
  - programming
title: "Ossec: The Great Makefile Refactor"
---

This was an idea for a blog post around 1.5 years ago when the OSSEC team
started the the great refactor, but sadly I did nothing with it till now. 

## OSSEC build that always worked, but made me mad

OSSEC has been using a cross platform build script written in sh for ages.
This system had worked for ages and worked everyplace most of the time.  See
that was the problem changes to the script were very error prone and always on
systems we did not have access to test on (I am looking at you AIX and HPUX).
Come to think about I don't think a single release did not cause an issue due
to the sh build script.

Another thing about the build system the sprawl.  To make changes you would
need to at least review all the following files: 

- `install.sh`
- `src/Config.Make`
- `src/InstallServer.sh`
- `src/InstallAgent.sh`
- `src/Makeall`
- `src/Makefile`
- `**/Makefile` (*at least 20 files)

This is a lot of places to check to made a change to the build.  Before you ask
yes you needed to look into the `install.sh` has it had grown some logic that
was not in other scripts.  

Given all this it easy to see why a change was needed, but the real reason I
hated this systems was the last file in the above list: `Makefile`.  We have
1000's of lines of logic in shell then still are using `make` incorrectly.  

Early in the talk about changing the build system one thing that came up as a
requirement to not change the default `install.sh` interface.  It's simple and
friendly and people love it over the standard `./configure && make && make
install`.  So 

## One Makefile to rule them all

I had done this once before years ago with [waf][wad.io] and that was not well
received in 2010, and while it's a great build system OSSEC should not need
python to compile. The disdain autotools/autoconf was very apparent when ever
it came up in discussion.  

The choice seamed clear once we started just require [GNU Make][gnu-make].
This gave us a huge amount of flexibility and features, and is available on
every platform.  While it's not default on all the platforms that OSSEC
supports, but we thought it was a safe bet that someplace someone had a
packages for installing [GNU Make][gnu-make].  

I started to work and the [first pull request][first-pr] and this started the
process of a day long hack-o-thon for the OSSEC team.  **This was single best
day of working on open source in my life**.  People just kept doing more and
feeding back and improving things so fast.  It was great to see and be a part
of.  

The new Makefile build does a lot of things in a single file and is very large,
but it handles everything in less code then the original system.  

[first-pr]: https://github.com/ossec/ossec-hids/pull/334/files
[wad.io]: https://waf.io/ 
[gnu-make]: https://www.gnu.org/software/make/

#### Build targets

OSSEC has a few major build targets:  server, local, agent, and winagent.  We
did not break this into two makefiles as their is so much overlap in code and
how they are built.  We decided that command like argument would required for
which binaries to compile. 

``` bash
$ make TARGET=server 
# or 
$ make TARGET=agent
```

Inside the makefile the logic is simple and straight forward.  As you can see
here: 

``` make
ifndef TARGET
  TARGET=failtarget
endif # TARGET

ifeq (${TARGET},agent)
  DEFINES+=-DCLIENT
  OSSEC_CONTROL_SRC=./init/ossec-client.sh
  OSSEC_CONF_SRC=../etc/ossec-agent.conf
endif

ifeq (${TARGET},local)
  DEFINES+=-DLOCAL
  OSSEC_CONTROL_SRC=./init/ossec-local.sh
  OSSEC_CONF_SRC=../etc/ossec-local.conf
endif


.PHONY: build
build: ${TARGET}
ifneq (${TARGET},failtarget)
  ${MAKE} settings
  @echo
  ${QUIET_NOTICE}
  @echo "Done building ${TARGET}"
  ${QUIET_ENDCOLOR}
endif
  @echo

```

We used the `ifndef TARGET` it set it to `failtarget` if not command line
argument was specified.  We figured always needing to define a target is a good
safe bet.

#### Features selection 

Adding features at compile time is not a command line argument.   This allows
for some simple logic to for adding defines and libraries for the build.  This
logic repeats a lot. 

``` make 

USE_INOTIFY=no 

ifeq (${uname_S},Linux)
    DEFINES+=-DINOTIFY_ENABLED
    OSSEC_LDFLAGS+=-lpthread
else

ifneq (,$(filter ${USE_INOTIFY},auto yes y Y 1))
  DEFINES+=-DINOTIFY_ENABLED
  ifeq (${uname_S},FreeBSD)
    OSSEC_LDFLAGS+=-linotify -L/usr/local/lib -I/usr/local/include
    OSSEC_CFLAGS+=-I/usr/local/include
  endif
endif

```

As you can see in the above snippet Linux always enabled INOTIFY and if you
have the library installed and are running FreeBSD you can enable it by running
`make TARGET=agent USE_INOTIFY=y`

#### Databases and library detection

The old database detection scripts had a lot of logic in them to get things to
work, and porting that over into GNU Make was/is the complex part.  

> Ok, stand back, I going to try boolean logic
> - Someone someplace about this code. 

``` make 
MI :=
PI :=
ifdef DATABASE

  ifeq (${DATABASE},mysql)
    DEFINES+=-DMYSQL_DATABASE_ENABLED

    ifdef MYSQL_CFLAGS
      MI = ${MYSQL_CFLAGS}
    else
      MI := $(shell sh -c '${MY_CONFIG} --include 2>/dev/null || echo ')

      ifeq (${MI},) # BEGIN MI manula detection
        ifneq (,$(wildcard /usr/include/mysql/mysql.h))
          MI="-I/usr/include/mysql/"
        else
          ifneq (,$(wildcard /usr/local/include/mysql/mysql.h))
            MI="-I/usr/local/include/mysql/"
          endif  #
        endif  #MI

      endif
    endif # MYSQL_CFLAGS

    ifdef MYSQL_LIBS
      ML = ${MYSQL_LIBS}
    else
      ML := $(shell sh -c '${MY_CONFIG} --libs 2>/dev/null || echo ')

      ifeq (${ML},)
        ifneq (,$(wildcard /usr/lib/mysql/*))
          ML="-L/usr/lib/mysql -lmysqlclient"
        else
          ifneq (,$(wildcard /usr/lib64/mysql/*))
            ML="-L/usr/lib64/mysql -lmysqlclient"
          else
            ifneq (,$(wildcard /usr/local/lib/mysql/*))
              ML="-L/usr/local/lib/mysql -lmysqlclient"
            else
              ifneq (,$(wildcard /usr/local/lib64/mysql/*))
                ML="-L/usr/local/lib64/mysql -lmysqlclient"
              endif # local/lib64
            endif # local/lib
          endif # lib54
        endif # lib
      endif
    endif # MYSQL_LIBS

    OSSEC_LDFLAGS+=${ML}

  else # DATABASE

    ifeq (${DATABASE}, pgsql)
      DEFINES+=-DPGSQL_DATABASE_ENABLED

      ifneq (${PGSQL_LIBS},)
        PL:=${PGSQL_LIBS}
      else
        PL:=$(shell sh -c '(${PG_CONFIG} --libdir --pkglibdir 2>/dev/null | sed "s/^/-L/g" | xargs ) || echo ')
      endif

      ifneq (${PGSQL_CFLAGS},)
        PI:=${PGSQL_CFLAGS}
      else
        PI:=$(shell sh -c '(${PG_CONFIG} --includedir --pkgincludedir 2>/dev/null | sed "s/^/-I/g" | xargs ) || echo ')
      endif

      # XXX need some basic autodetech stuff here.

      OSSEC_LDFLAGS+=${PL}
      OSSEC_LDFLAGS+=-lpq

    endif # pgsql
  endif # mysql
endif # DATABASE

```

Yeah we know, but this includes all the old logic that the scripts system
preformed and some new ones. 

If anyone knows of a better way we would love to hear it ;) 

## Today 

I in writing up this Blog post I went back end reviewed the changes to the
makefile over the last year and 1/2, and not major changes to structure or
logic have happened.  Just new features and improvements, and looks like 
a solid foundation.  









