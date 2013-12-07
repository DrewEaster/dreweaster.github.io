---
layout: post
title: "What a storming idea"
date: 2013-12-07 09:46
comments: true
categories: [DDD, Event Storming, EDA]
---
Firstly, an important disclosure - having only just stumbled across the practice of [Event Storming](http://ziobrando.blogspot.be/2013/11/introducing-event-storming.html), I’ve not yet had the opportunity to experiment with it myself. However, sometimes a concept/practice makes such immediate sense in one’s mind, that one feels compelled to talk about it!

This post feels like quite a natural successor to my previous post on [Event-driven Architecture](http://www.dreweaster.com/blog/2013/12/02/event-driven-architecture-ftw/). In that post, I discussed some of the tangible benefits of EDA, and this follow up introduces the practice of [Event Storming](http://ziobrando.blogspot.be/2013/11/introducing-event-storming.html), a fledgling Domain Driven Design influenced practice that goes hand in hand with EDA to assist product teams in exploring the complexity of their business domain.

It’s not my intention to write a long article on Event Storming - I encourage you to read Alberto Brandolini’s  [introduction](http://ziobrando.blogspot.be/2013/11/introducing-event-storming.html) instead - but I want to share my biggest takeaways from this new learning.

The Domain-Relational Impedance Mismatch
----------------------------------------

In my experience of domain modelling, the biggest mistake I’ve seen made, time and time again, is to fail to involve the most important people ‘in the room’. The reason for this is that far too many projects take a _persistence oriented_ approach to modelling, as opposed to a _business oriented_ approach. Put simply, by tending to address model genesis using only technical team members, teams get sucked into allowing technical persistence concerns to shape their domain model. It’s not unusual for domain experts to be involved only at the user interface level, leaving software engineers and DBAs to make decisions that they are probably the least qualified to be making. The ultimate success of a software project resides on how well domain complexity has been tackled, and so it seems crazy for domain experts to be absent from the modelling process.

Let’s be absolutely clear - a domain model exists in spite of persistence technology, not because of it. A model is not a technical concept; it is a reflection of a business domain that exists in real-life, not inside a machine.

Whilst NoSQL technology is becoming more popular, it’s still fairly normal to see domain modelling tackled using entity relationship (ER) diagrams - something quite familiar to engineers and DBAs. That wily old fox, the relational model, is still recognised by many as the de facto way to practice domain modelling. However, Domain Driven Design (DDD) teaches us a much better way to address modelling, and does not make room for persistence concerns in modelling conversations - models spawned using DDD practices typically appear very different to what they’d look like had ER modelling been applied instead.

You’re probably familiar with the [object-relational impedance mismatch)[http://en.wikipedia.org/wiki/Object-relational_impedance_mismatch] concept, but I think our problems extend much further than that; I believe DDD teaches us of a domain-relational impedance mismatch. That is, the relational model is not a natural fit for addressing domain model complexity, and thus should not be trusted to do so.

The light at the end of the tunnel
----------------------------------

So, we know we're doing it wrong, but how do we then ensure we get both the right people in the room (domain experts), and a method to support effective communication of domain complexity? This is where I believe we should look to Event Storming to help us out.

From my initial learnings on Event Storming, I can wholeheartedly say that this technique appears to offer a very attractive way to ensure focus remains on the business rather than technical implementation. It forces the right people - domain experts - to be in the room, thus ensuring core business flows are identified, bounded contexts are defined and consistency boundaries are clarified.

Event Storming does infer an event-driven architecture (EDA), but I hope my [previous post](http://www.dreweaster.com/blog/2013/12/02/event-driven-architecture-ftw/) serves to address why you should be doing that anyway. It finally gives us an accessible technique that allows domain experts and technical specialists to work together to tackle domain complexity effectively. It’s a really exciting prospect and I look forward to applying it both in existing and future projects.
