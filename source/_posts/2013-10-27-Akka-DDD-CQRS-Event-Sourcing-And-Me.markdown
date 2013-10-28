---
layout: post
title: "Akka, DDD, CQRS, Event Sourcing and Me"
date: 2013-10-27 20:47
comments: true
categories: [Akka, Scala, DDD, CQRS, Eventsourcing]
---
Introduction
------------

I've been involved in some really interesting discussions on the Akka User mailing list of late and thought it would be good to translate something I wrote, in one particular thread, into a blog post.

I recently became rather obsessed with Domain Driven Design (DDD), CQRS and Event Sourcing. Given an existing passion for Akka (actor based programming), it dawned on me that the actor model might be an extremely good fit the aforementioned "Holy Trio" of paradigms/patterns. Whilst trying to build a prototype, I brought up the topic on the Akka User mailing list and became aware that the Akka Team were actively working on Akka Persistence, which is well ahead of my dirty little prototype! For one reason or another, within a post on the mailing list, I ended up brain dumping my understanding of how Akka Persistence and the "Holy Trio" fits together, so here follows a blog friendly version!

Akka and the Holy Trio
----------------------

So, it seems like the actor model fits very well with DDD, and I'm not the first person to think this - [Vaughn Vernon](https://vaughnvernon.co/) is well ahead of the game!

It feels quite natural to me to consider an actor analogous to an aggregate root (in DDD speak). But, rather than just seeing an actor as a mediator sitting in front of a database, you see the actor conceptually as an entity. By incorporating event sourcing, the actor works on the basis of processing a command (representing some action) and outputting one or many events that represent the result of processing that command. At all times the actor encapsulates its current state, which can simply be seen as the result of a series of ordered events applied to it.

Initially, it's actually quite helpful to ignore persistence requirements altogether and just rely on the state within the actor - in this way you don't allow persistence concerns to influence the way you design your domain - I often find that domain models end up messy because, despite attempts to avoid doing so, persistence concerns end up, in one form or another, leaking into the business logic. Any approach to building a domain layer, that can genuinely remove the need to consider persistence technologies, is an attractive proposition in my mind!

By following an event sourcing approach, persistence becomes no more complex than just persisting the events - that the actor produces - in an event store (journal). Given that events represent an immutable history of state changes, the journal can be append only, and that's pretty exciting from a performance perspective. This is something that can quite clearly be bolted on once you've already put together a working in-memory only version of your domain - the actors themselves can be completely agnostic to the existence of the persistence mechanism. This is where Akka Persistence comes in - it works on the following basis:

1. Actor receives command
2. Actor processes command (applies business logic) and produces events that represent results of processing
3. Events are persisted to journal
4. Events are applied to actor so it can update it's internal state

Akka Persistence pretty much deals with everything other than the command handling step, which represents your custom business logic - Akka is not quite good enough (yet) to do that bit ;-)

Step 4 is quite important - it's key that internal state is only applied once you're sure events have been persisted to the journal. Also, by separating this step, it serves the additional purpose of allowing events in the journal to be replayed onto a "green" instance of the actor such to bring its internal state back up to current. Akka Persistence also has support for "snapshots" which allow you to take periodic snapshots of current state to be used as a performance optimisation when replaying events.

So, to sum up, the actor model just fits with the "Holy Trio". It seems to deal with a majority of the pain points experienced when building a domain layer using more traditional CRUD techniques. It's pretty exciting to me how natural this design feels without any need to consider using an ORM ;-)

The Future
----------

And that's the end of the blog friendly version of my brain dump! I've been involved in further discussions on the mailing list, including collaborating with Vaughn Vernon on his vision for a simplified model for DDD in Akka. I hope to keep involved in these discussions as I truly believe the combination of the actor model with DDD, CQRS and Event Sourcing is going to become a very prevalent domain model design solution in the future.