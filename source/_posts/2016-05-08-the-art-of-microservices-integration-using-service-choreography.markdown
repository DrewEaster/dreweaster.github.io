---
layout: post
title: "The art of microservices integration using service choreography"
date: 2016-05-08 22:10
comments: true
categories: [microservices, event-driven-architecture]
---
This is the first post in a two part series looking at the topic of microservice integration. In this first instalment, I'll be focusing mainly on the theory side of event-driven service choreography. In the second part, I'll cover practical application of the techniques discussed in this first post, looking at the pros and cons of specific implementation technologies. So, let's get on with part one!

Looking back
------------

One of the biggest shortcomings of traditional SOA is/was the tendency to break up a highly-coupled monolith into a series of smaller services with the same level of coupling that was previously internal to the monolith. The likely result being a distributed monolith with all the same problems you had before, but now with an additional operational burden - you end up in a worse position than if you'd just stuck with the monolith!

It's this learning that I believe was the main catalyst for the development of the microservices architecture pattern (SOA 2.0?). In hindsight it seems pretty obvious; if you can't run a service in isolation, with significant levels of autonomy, it's pretty hard to justify why a piece of functionality is better in a separate service than simply internal to a monolith. It's not like there aren't some potentially good reasons why people would choose to do old style SOA, it's just you find those good reasons are outweighed by the negatives most of the time.

Looking forward
---------------

So, enter the world of microservices architecture, and the promise of isolation, autonomy, single-responsibility, encapsulated persistence etc. But, what exactly allows one to achieve such isolation and autonomy?

In this post, I'm going to focus on how getting service integration right is a fundamental part of what enables services to be isolated, and act autonomously. I believe having a thorough understanding of the principles of good service integration can train your mind into grasping the importance of good service separation. As a little bonus, I'll close this post by looking at some considerations for good service separation.

Service orchestration vs service choreography
---------------------------------------------

In a classic distributed monolith scenario as described above, the prevalent integration technique is likely to involve service _orchestration_. This is where backend services typically have a high-level of synchronous coupling - i.e. a service is reliant on other services to be operational and working, within a single request/response cycle, in order for it to carry out its own responsibilities. Such real-time dependencies prevent a service from acting autonomously - any failures in its dependencies will inevitably cascade, preventing the service from fulfilling its own responsibility. Here's a visualisation of service orchestration in action:

[{% img center /images/service_orchestration.png %}](/images/service_orchestration.png)

In this example, service A is dependent on both service B and service C when handling its own inbound HTTP calls. Service A fetches state X (from service B) state Y (from service C), aggregating them with some of its own state, Z, to finally yield an HTTP response to its callers. Failures in either B or C will prevent A from fully fulfilling its responsibilities - it may be able to degrade gracefully but the ability to do so really depends on context.

Contrast this scenario with service _choreography_ instead. In a system designed to embrace choreography, services will typically avoid synchronous coupling - i.e. any integration between services does not apply during the usual request/response cycle. In such cases, a service can fulfill its responsibilities within the request/response cycle without the need to make further calls to other services (with the exception of persistence backends owned solely by the service).

The classic way to achieve this is via embracing an event-driven (message passing) approach to integration. That is, any state that a service requires from services external to it, is projected internally from event streams published by those external services. Such internal projections will be managed and updated completely asynchronously (outside of the request/response cycle), and will be eventually consistent. In the true spirit of microservices, the service entirely encapsulates all the persistent state it requires in order to fulfill its responsibilities, achieving true isolation and autonomy.

Let's refactor the previous diagram to reflect choreography instead of orchestration:

[{% img center /images/service_choreography.png %}](/images/service_choreography.png)

In this updated approach, service A receives events from streams published by service B and service C. Service A processes events in an eventually consistent manner, and persists locally only what it needs of state X and Y, alongside its own state, Z. When handling an incoming request, service A does not need to communicate with service B or service C.

Clarifying asynchronous integration in service choreography
-----------------------------------------------------------

I feel there's some confusion where people refer to asynchronous communication, especially in the field of microservices integration. It's worth some time to clarify what's meant in the context of service choreography.

What it's not:

* **Non-blocking I/O** - I'm an advocate of asynchronous, non-blocking I/O as part of building more efficient, resilient and scalable services that interact in some way with external I/O. However, in the context of service choreography, this is certainly not what we mean by asynchronous integration. Non-blocking I/O could still be used within the request/response cycle for orchestration use cases, and, whilst it has its advantages in one sense, certainly doesn't, on its own, buy any architectural benefits of isolation and autonomy.

* **Classic MQ Request/Reply** - It's possible using classic MQ technology (e.g. JMS, AMQP) to achieve asynchronous request/reply behaviour. You could pop a message on a queue, and wait for a response on some temporary reply queue. There's certainly some added decoupling in that the caller needn't know exactly who will reply, but, like with non-blocking I/O, if this is being done as part of a service handling an incoming request, then, despite the communication with the MQ itself being asynchronous in nature, the service is still not acting autonomously. If a consumer responsible for replying is down, and the call must then timeout, it's ultimately no different to an HTTP endpoint being unavailable or failing.

So, to clarify things then. When we're talking about services being integrated asynchronously as part of service choreography, we're referring to a service being _free from the need to rely on its dependencies during the request/response lifecycle_.

End-to-end autonomy
-------------------

Where I've covered isolation and autonomy in this post, I've been referring to _runtime_ autonomy. It's worth noting that a strong motivation for isolating services is the autonomy they additionally afford throughout the entire engineering process. The event-driven integration nature of service choreography sits very naturally with the desire to assign clear ownership to specific teams, enabling them to develop, build, test and release independently. Whilst there are techniques to support these things alongside service orchestration, I find it much easier to reason about isolation and independence when service dependencies are largely confined to the background.

When considering service/API level tests, embracing eventually consistent, event-driven integration allows you to focus on data fixtures as opposed to service virtualisation. There's something inherently more simple - in my mind - about placing a system into a desired state via a simulated event stream, rather than having to worry about mocking/stubbing/virtualising some external service endpoint(s).

Added complexity?
-----------------

Service choreography, without doubt, introduces a level of technical complexity beyond orchestration. Rather than the apparent simplicity of making calls to dependencies in real-time, you need to both produce events yourself and consume events from others; it requires a fundamental shift in the way you build software. It exposes you to such challenges as guaranteeing at-least-once-delivery/processing of events, handling out of order events, ensuring idempotency in event consumers, factoring in eventual consistency in the UI etc.    

Like with anything in software engineering, it's all about tradeoffs. There's no silver bullet, and you have to make a call based on your own unique circumstances. There will always be times where the simplicity of orchestration will trump its limitations on autonomy. Making such calls is why experience matters and why there will always be room for judgement in engineering.

For example, a team may decide that it's ok for services acting as companions to some primary service - and so invisible outside the context of the service's public contract - to use orchestration, reserving choreography for integrating with services owned by other teams. In this scenario, the team would still benefit from the greater independence they'll have from other teams, whilst losing some runtime autonomy internally.

Additionally, there are times when integrating with third-party services (outside your organisation) will necessitate a degree of orchestration given limitations in the third-party API contracts.

As a general system wide architectural constraint, it's wise to be rigid about enforcing service choreography between services owned by different teams. This has the added benefit of really helping to drive home the importance of good service boundaries - if your design relies on orchestration between services owned by different teams, there's a good chance you've not found sensible business-oriented boundaries between your services. A word of caution, though: where services are clearly aligned with business boundaries, choreography should be the preferred approach, even if the services are owned by the same team.

Touching on boundaries
----------------------

Whilst service choreography enables autonomy, the elephant in the room is that a dependency is still a dependency, whether it be event-driven or not. If a service is required to consume from a large number of event streams, it's still adding overhead in terms of managing the dependencies over time.

One of the fundamental mistakes people make with microservices is to draw technical boundaries rather than boundaries of business capability/purpose. The use of the word "micro" is surely partly to blame, as it may appear to encourage highly granular service responsibilities. By breaking up a monolith into services of a highly technical nature, with little correlation to the business domain, you're inevitably going to introduce more dependencies, and accordingly more overhead. Too many dependencies of any variety is almost certainly a design smell, a sign of high coupling and low cohesion, when it's the exact opposite we're looking for. If you're stuck with lots of dependencies, it's a sure fire way of spotting that you've drawn your boundaries wrong. If you've managed to identify a genuine business capability, you'll be surprised at its fairly natural properties of isolation and independence.

Domain-Driven Design equips us with the toolset - and mindset - to identify business-oriented boundaries, and there's certainly reason to see a close relationship between microservice boundaries and Bounded Contexts as described in DDD. Whilst I don't consider there to be a direct 1:1 mapping in every circumstance (a good subject for another post), there are definitely some parallels to draw.

One way to go about reasoning about the need to introduce a dependency on an external event stream is to pedantically question the purpose of having that dependency in the first place. You may begin to find that you're tending to introduce such dependencies for the purpose of presenting data in a UI rather than for fulfilling specific business logic within your service boundary. In such cases, it can be preferable to rely on data aggregation/composition in the UI layer (where it's generally a little more acceptable/inevitable to rely on orchestration). When applying Domain-Driven Design to model business complexity, it's advisable to be ruthless about business purpose within any Bounded Context, and that means avoiding projecting state from external services if your service/domain doesn't _really_ need it.

Coming next
-----------

That wraps up part one of this two part series. I'll be following up soon with part two within which I'll be evaluating the suitability of specific implementation technologies. Stay tuned!
