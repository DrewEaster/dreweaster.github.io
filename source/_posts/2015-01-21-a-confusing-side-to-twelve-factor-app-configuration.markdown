---
layout: post
title: "A confusing side to twelve factor app configuration"
date: 2015-01-21 18:39
comments: true
categories: 
---
I'm sure I'm not the only one who breathed a sign of relief when the clever brains behind Heroku published the [Twelve Factor App](http://12factor.net) guidelines. Here was a reliable set of principles - born out of real experience - that immediately hit home, providing sensible advice for overcoming the many pitfalls developers and operations teams have fought relentless battles with time and time again.

One of the principles that I was especially able to connect with is the advice regarding from where applications should read their [configuration](http://12factor.net/config). Having regularly encountered config file hell over the years, this simple, platform agnostic approach to supplying config to an application makes a whole lot of sense. There have, though, been some misunderstandings with regard to this advice, [and this blog post](http://blog.doismellburning.co.uk/2014/10/06/twelve-factor-config-misunderstandings-and-advice/) by Kristian Glass does a good job of highlighting one such misunderstanding - Twelve Factor does not say how the environment should be populated, only that an app should read from it.

So, with a major misunderstanding out the way, we are left with a whole bunch of options as to how we populate the environment. Outside of the extreme abstraction of a PaaS environment like Heroku, something like Consul - the distributed key/value store - is one such solution. And in this [blog post](https://hashicorp.com/blog/twelve-factor-consul.html) by [Hashicorp](https://hashicorp.com), that very solution is covered quite nicely.

However, I'd like to challenge Hashicorp's claim that this approach is compatible with Twelve Factor principles. My personal opinion - on which I'm very happy to be challenged - is that there is a further, quite critical, piece of advice, regarding configuration, contained within the Twelve Factor guidelines - you just have to turn to a different page.

 This additional piece of advice is contained with the section titled [Build-Release-Run](http://12factor.net/build-release-run). As far as I'm concerned, the advice here is pretty crystal clear - a 'Release' is a combination of a 'Build' _and_ 'Config'. This _immutable_ artifact is to be versioned and is deployed and rolled back as an atomic unit. This is how Heroku does it, and the open source (Docker)[http://www.docker.com] based PaaS [Deis]()


