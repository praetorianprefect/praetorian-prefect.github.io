---
title: RickRoll Everyone with BlueCoat
date: 2009-03-16
categories: 
  - networking
tags: 
  - bluecoat
  - youtube
  - proxy
  - networking
series:
  - "Praetorian Prefect"
slug: RickRoll-Everyone-with-BlueCoat
aliases:
  - /RickRoll-Everyone-with-BlueCoat.html 
  - /rickroll-everyone-with-bluecoat.html
author: Jeremy Rossi
description: |
    The [Bluecoat SGOS](http://www.bluecoat.com/products/sg) can do a fair amount 
    of stuff just like any web-proxy should, but my favorite is to 
    [RickRoll](http://en.wikipedia.org/wiki/Rickrolling) the whole 
    company.  ( _People spend to much time on youtube as is_ ).
---

### BlueCoat Proxy ###

The [Bluecoat SGOS](http://www.bluecoat.com/products/sg) can do a fair amount 
of stuff just like any web-proxy should, but my favorite is to 
[RickRoll](http://en.wikipedia.org/wiki/Rickrolling) the whole 
company.  ( _People spend to much time on youtube as is_ ).

In this example users are authenticated with NTML back ended by Windows
Active Directory. See the docs from Bluecoat on how to set this up.

#### Definitions Conditions ####

Conditions allow you to control when things should happen. They do
nothing by themselves, but get put together later to preform some real
fun.

The first definition here matches only member of the group
`DOMAINpxy_rickrolld`. You could make this users or just about that
think you would like. I choice the group method to make it simple to add
and remove effected users.

```bash
define condition group_to_be_rickrolled realm=active_directory
  group=DOMAIN\pxy_rickrolld end
```

The second definition just matches does a REGEX to match the domain
"`youtube`" and looks for the string "`watch`" in the url path. The use
of REGEX really is not the best way to do this, but I figured showing
both methods of matching was worth the slight performance hit.


```bash
define condition match_url_to_rickroll
    url.host.regex="youtube" url.path.substring=watch 
end
```


#### Definitions Actions ####

Actions are define something to do with a request. In this case
we are going to rewrite the request and change the video to the
"`oHg5SJYRHA0`".


```bash
define action youtube_change_to_rickroll
    rewrite( url, "(http://.*/watch?v=)([^&]+)(.*)", "$(1)oHg5SJYRHA0$(3)" )
end
```

Given the initial url of "`http://www.youtube.com/watch?v=OBghD0XBN5M&feature=related`".  

The rewrite functions second argument is a REGEX that stores the following:

* "`http://www.youtube.com/watch?v=`" in variable "`$(1)`".
* "`&feature=related`" in variable "`$(3)`".  

The third argument is the Newly created url that simply puts the data
back together with our selected Video ID.

#### Proxy Section ####

Now that you have everything defined you need to put it all to use.   

```bash
<Proxy> 
    condition=match_url_to_rickroll condition=group_to_be_rickrolled 
action.youtube_change_to_rickroll(yes) 
```


This will pull all the define from above to select when to preform the
rewrite function. Putting this in place is fun, but it really does make
people mad for some reason.


### Completed Fun ###


```
define condition group_to_be_rickrolled
    realm=active_directory group=DOMAIN\pxy_rickrolld 
end
define condition match_url_to_rickroll
    url.host.regex="youtube" url.path.substring=watch 
end	
define action youtube_change_to_rickroll
    rewrite( url, "(http://.*/watch?v=)([^&]+)(.*)", "$(1)oHg5SJYRHA0$(3)" )
end

<Proxy> 
    condition=match_url_to_rickroll condition=group_to_be_rickrolled action.youtube_change_to_rickroll(yes) 
```


### Results ###

Well of course it had to be done.

<object width="425" height="344"><param name="movie" value="http://www.youtube.com/v/oHg5SJYRHA0&hl=en&fs=1"></param><param name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed src="http://www.youtube.com/v/oHg5SJYRHA0&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed></object>



