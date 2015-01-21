---
layout: post
title: "A confusing side to twelve factor app configuration"
date: 2015-01-21 18:39
comments: true
categories:
---
I'm sure I'm not the only one who breathed a sign of relief when the clever brains behind Heroku published the [Twelve Factor App](http://12factor.net) guidelines. Here was a reliable set of principles - born out of real life experience - that immediately hit home, providing sensible advice for overcoming the many pitfalls developers and operations teams have fought relentless battles with time and time again.

One of the principles that I was especially able to connect with is the advice regarding from where applications should read their [configuration](http://12factor.net/config). Having regularly encountered config file hell over the years, this simple, platform agnostic approach to supplying config to an application makes a whole lot of sense. There have, though, been some misunderstandings with regard to this advice, [and this blog post](http://blog.doismellburning.co.uk/2014/10/06/twelve-factor-config-misunderstandings-and-advice/) by Kristian Glass does a good job of highlighting one such misunderstanding - Twelve Factor does not dictate from where the environment should be populated, only that an app should read from it.

So, with a major misunderstanding out the way, we are left with a whole bunch of options as to how we populate the environment. Outside of the extreme abstraction of a PaaS environment like Heroku, something like Consul - the distributed key/value store - is one such solution. And in this [blog post](https://hashicorp.com/blog/twelve-factor-consul.html) by [Hashicorp](https://hashicorp.com), that very solution is covered quite nicely.

But, hang on, has something been overlooked?

I'd like to challenge Hashicorp's claim that this approach is compatible with Twelve Factor principles. My personal opinion - on which I'm very happy to be challenged - is that there is a further critical piece of advice, regarding configuration, contained within the Twelve Factor guidelines - you just have to turn to a different page.

This additional piece of advice is contained with the section titled [Build-Release-Run](http://12factor.net/build-release-run). As far as I'm concerned, the advice here is pretty crystal clear - a 'Release' is a combination of a 'Build' _and_ 'Config'. This _immutable_ artifact is uniquely versioned and is deployed and rolled back as an atomic unit. This is how Heroku does it, and the open source [Docker](http://www.docker.com) based PaaS [Deis](http://www.deis.io) has the [same approach](http://docs.deis.io/en/latest/reference/terms/release/#release). I'm leaving out other PaaS tech that also follows this model.

It would be hard to deny that this approach makes deployments very easy to reason about. By bundling together code alongside environment specific config, you simplify release management and tracking. For example, if you need to rollback, you don't have to manage that as two separate actions - it's an atomic action and will return you to the last known uniquely versioned, operationally sound combination of code and config.

So, whilst it's true that Twelve Factor does not dictate from _where_ the environment should be populated, it's pretty clear that it does say _when_ it should be. The source of environment configuration should be read ***when preparing a release***, and that any change to config, just like with a change to code, ***results in a new release***.

Therefore, any approach to managing configuration, in a similar way to Hashicorp's advice, would not appear to be compatible with the Twelve Factor guidelines. They are promoting a model where the 'Build' and 'Config' are not atomically bundled as a 'Release' and, as such, this model is violating a fundamental Twelve Factor principle - if your code and config is managed in separate lifecycles, it ain't Twelve Factor compatible.

One could quite easily challenge the 'Build + Config = Release' advice:

1. It doesn't appear to leave any room for runtime config changes
2. It's not completely clear how it would work with dynamic service discovery

Sometimes, however, the advantages of predictable, easy to reason about deployments outweigh the benefits of such niceties

I'm not debating here what's the right or wrong way, I'm just pointing out that the Twelve Factor advice is very clear about the meaning of a 'Release', and, therefore, any method that circumvents this, cannot claim to be compatible with the guidelines.

Just saying.

### UPDATE 2015-01-21 20:10

Spring are also claiming that the [Spring Cloud](http://projects.spring.io/spring-cloud/spring-cloud.html) approach to configuration is Twelve Factor compatible in this [blog post](http://spring.io/blog/2015/01/13/configuring-it-all-out-or-12-factor-app-style-configuration-with-spring?utm_content=bufferfa5a5&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer). I think this is making the same mistake as Hashicorp. If configuration is able to change independently from a release - as is described in this post - then it's not in the spirit of the Twelve Factor App.
