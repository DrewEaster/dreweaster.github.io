---
layout: post
title: "Event-driven Architecture FTW"
date: 2013-12-02 21:41
comments: true
categories: [EDA, Eventsourcing, Architecture, Integration, DDD]
---
A quick primer on EDA
---------------------

First of all, let's delegate to the all-knowing source, [Wikipedia](http://en.wikipedia.org/wiki/Event-driven_architecture), to give us a concise definition of Event-driven architecture (EDA):

> "Event-driven architecture (EDA) is a software architecture pattern promoting the production, detection, consumption of, and reaction to events."

It's pretty unusual to encounter a software engineer who hasn't dabbled with publish-subscribe semantics - or anything resembling the [Observer Pattern](http://en.wikipedia.org/wiki/Observer_pattern) - at some point in their engineering adventures. It's hard to ignore, when faced with a challenge to simplify some programming problem, the draw of a decoupling a producer of something from one or many consumers of that something. Not only does it divide responsibility (making code easier to test for starters), it also clears the way for an asynchronous programming model in use cases where producers needn't be blocked by the actions of an interested consumer.

If you can appreciate the power of publish-subscribe, you're unlikely to have much trouble in figuring out how Event-driven architecture (EDA) could help you. It only requires a small leap of faith to realise that pattern you used to solve that micro problem, in some isolated component within your system, can become a macro pattern to underpin a system-wide architectural style.

So, let us return to that Wikipedia definition. I think we could reasonably interpret that definition to understand EDA as the art of designing a system around the principle of using events - concise descriptions of state changes or significant occurrences in the system - to drive behaviour both within and between the applications that make up an entire platform.

### Production
Production of an event involves some application component/object generating a representation of something that has happened - maybe a state change to an entity, or maybe the occurrence of some activity (e.g. a user viewed page X). Rather than notifying specific consumers, the producer will simply pass the event to some infrastructural component that will deal with ensuring the event can be consumed by anything that's interested in it.

### Detection
It's my belief that the Wikipedia definition is essentially referring to the mechanism that sits between a producer and consumer (the infrastructure piece) - the logic that ensures events are passed to interested consumers.
§
### Consumption
Consumption is the act of an interested consumer receiving an event for which it is interested in. This is still most likely a part of the infrastructure piece (e.g. a message bus). There might be a whole bunch of stuff here about reliable delivery, failure recovery etc.

### Reaction
Reaction is the fun part where a consumer actually performs some action in response to the event that it has consumed. For example, imagine you register a user on your website and you want to send them a welcome email. Rather than bundling the responsibility for sending email within your domain model, just create a consumer to listen into a UserRegisteredEvent and send an email from there. This nicely decouples the email delivery phase and, also, allows it to be done asynchronously - you probably don't need, or want, email delivery to be a synchronous action. Also, imagine you have further requirements relating to post registration behaviour - your domain model would soon become unwieldy with all that extra responsibility. Not one to want to violate the Single Responsibility Principle (SRP), you sensibly use event programming to separate all those actions into separate consumers, allowing each behaviour to be tested in isolation, and retain simplicity in your domain model.

Events everywhere
-----------------
As previously alluded to, it's far from unusual to see fragments of event programming in many applications. However, it's another step entirely to see event programming adopted in such a way that it becomes an endemic architectural pattern - that is, where an entire platform uses events to underpin all its moving parts. To be successful with EDA, it needs to become a fundamental mindset that drives all design decisions, rather than just a pattern that used in some isolated parts of a wider system.

I want to evaluate a series of application/platform features that, whilst might sit outside of core business workflows, fit really nicely with EDA. This should help to realise why EDA can be a very fruitful path to follow.

### WebHooks (the Evented Web)
WebHooks is a fairly high level concept that encompasses the use of HTTP as a form of publish-subscribe mechanism. Whilst there are more recent attempts to create more standards around this - [calling it the Evented Web](http://progrium.com/blog/2012/11/19/from-webhooks-to-the-evented-web/) - the fundamental idea is to allow a consumer to register a callback URL with some remote resource (using HTTP) that will be invoked by the hosting service whenever some event occurs relating to that resource. A really well known example is post-commit hooks on Github - any external tool (e.g. a CI server, a bug tracker) can register interest in commits to a repository and react in whatever way that makes sense in their context.

It's my belief that the Evented Web paradigm has got a lot of growth potential and will become a defacto expectation of any well designed service API. What's clear, I hope, is how easy it would be to add WebHooks functionality to your own application if you are already applying EDA across the board.

### Big Data
I do kind of detest using the term "Big Data" as it's uncomfortably ambiguous and vague. However, for the purposes of this article, I'm going to stick with it (strike me down). If, for now, we think of Big Data as a way of capturing shed loads of data to enable business intelligence, we should be able to see quickly that events occurring within an application might be a great source of all that lovely data. If you've adopted EDA, your Big Data pipeline may just be a consumer of all your application events. You might dump all those events into HDFS for future batch processing, and, given you are essentially subscribing to a real-time event feed, you might also drive your real-time analytics use cases as well.

### Monitoring
Unless you are someone who really couldn't give a damn, you're going to want some monitoring tools in place to give a thorough insight into the health of your system in production. Common monitoring solutions may include, amongst other things, a bunch of smart dashboards full of sexy graphs, and some threshold alerting tools to help spot potential problems as soon as an incident starts. Either way, both these tools are driven by time series data that represents stuff happening within an application. What better way to capture this 'stuff' than sucking events in from your event streams? Again, if you've already followed EDA, you're going to get some pretty quick monitoring wins.

### Log file analysis
There is possibly some crossover here with monitoring, and even Big Data, but I think it deserves its own special mention. If you imagine logs as comprehensive streams of events, if you've followed an EDA style, you can pretty much get log analysis for free. Just suck your logs into some analysis tools (e.g. [Logstash](http://logstash.net/) and [Kibana](http://www.elasticsearch.org/overview/kibana/)), and you're pretty much good to go. Just remember that it's perfectly reasonable to use events to represent errors too (which could contain any relevant stack trace).

### Test-driven Development (TDD)
Okay, so TDD is not an application feature, it's part of the engineering process. However, if our architecture decisions can help to improve our quality process, then that can't be a bad thing. Event driven programming encourages a code level design approach that follows the [Tell, Don't Ask](http://pragprog.com/articles/tell-dont-ask) pattern. You tell an object to do something, which leads to an event, or events, being published. So, what's this got to do with TDD? In my experience, it's much easier to reason about your code, and define more coherent specifications, if your testing style mimics "given this input (command), I expect this event, or events, to be produced". A test first approach is very compatible with this style, and makes you think in a much more behavioural (think BDD) way.

For the win
-----------
Right, we've covered a primer of EDA and seen how it can be used to drive both core business flows, cross cutting concerns, and even our quality process. I believe this knowledge makes a very compelling case for adoption of EDA - why would you bake in custom solutions for capturing Big Data, doing health check monitoring etc, when you can simply piggy back these features off of your core architecture? Hopefully, you wouldn't! All sorts of wonderful acronyms pop into my head at this point - KISS, DRY, SRP etc. And don't we all love acronyms?

But can we go even further?

Going the whole hog with event sourcing
--------------------------------------
This discussion leads so elegantly into the final part of this blog post - event sourcing. Event sourcing is an approach to persistence that means the state of an entity - more specifically, an Aggregate Root in DDD speak - is made up of all the events, representing state changes, it has emitted over time. So, rather than than store current state in a database, you simply load the historical sequence of events (from an event store) and apply them in order to obtain current state. I will leave it up to the reader to pursue the full benefits of using event sourcing, but here are some of the headline wins:

- Supports a simple, and very scalable approach to persistence. An event store can be as simple as an append only log file.
- Gives you a full history of every state change and that's great for producing an audit log (something you might want anyway, even without event sourcing).
- Can still utilise snapshots of current state for performance optimisation when replaying
- Very compatible with a test first, behavioural approach to testing.
- Plays very nice with the [CQRS](http://martinfowler.com/bliki/CQRS.html) architectural pattern, a very practical way to bake scalability into your applications by maintaining separate paths for reads and writes.

If you're going to go down the EDA route, why limit just your applications to an event driven style? If it's possible to maintain state via the events you're already publishing, why maintain a separate database on top? I think this is a really fascinating area of exploration - sometimes traditional CRUD might be a better choice, but the more I work with event sourcing, and the more comfortable I feel with EDA in general, the harder it becomes to find good reasons not to follow this path.

Conclusion
----------
So that wraps up a fairly lengthy discussion on EDA and how an event driven mindset can promote a coherent strategy for the way you build software. One of the toughest things we face as software engineers is maintaining a consistency in style, especially in large teams. For me, EDA is an accessible and clearly defined way of thinking and, if you can introduce your whole team to that mindset, it will help to bridge the gap between doing the right thing (building product features your users love), and doing the thing right (consistent and elegant technical solutions).





