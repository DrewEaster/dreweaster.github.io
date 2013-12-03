---
layout: post
title: "The fallacy of the omniscient domain model"
date: 2013-11-17 17:20
comments: true
categories: [DDD]
---

Complexity. As software engineers, it's pretty hard to make it beyond lunch without someone mentioning it. But, what is it exactly? Most of us probably think we sussed this out a long time ago - hell, we're probably preaching the KISS and DRY principles on a daily basis. However, complexity in software is a multi-faceted beast and, with that in mind, taking the time to reflect on your own view of complexity, you may reveal a bunch of defective preconceptions.

One of these preconceptions, one which I've failed to question effectively myself in the past, is that a 'simple' domain model is one that can encompass the entire domain of an enterprise. The other preconception is that 'simple' architecture means few moving parts.

Let's address each one individually.

Imagine a completely DRY, one-size-fits-all domain model that manages to perfectly model the domain of an entire enterprise. This model is infinitely malleable and able to accommodate future changes without adversely affecting existing code dependant on it. Are you struggling to imagine this? I hope so, because I don't think it's possible in anything other than the most basic of business domains. Regardless, this a very common approach and, instead of being simple, it adds an alarming amount of overall complexity. Ultimately, the intertwining of vaguely related entities makes it impossible to make changes in one place without having to untangle deep dependencies elsewhere.

If, instead, you apply some of the core principles of Domain Driven Design (Domains, Subdomains, Bounded Contexts) to any enterprise, it's natural to materialise multiple models that exist within the different subdomains and contexts that make up the wider domain. This approach, which actually reflects the structure of the real business, reduces overall complexity - dependencies are untangled by design, making changes more achievable in isolation.

One important thing to note here - you're not necessarily violating the DRY principle if an entity appears in multiple contexts. Maybe you've just failed to successfully tweak the Ubiquitous Language for each context, such that you're recognising that entity to mean different things in those different contexts.

So, now onto our second preconception - does simple architecture mean few moving parts? My previous argument may seem overly convenient given it implicitly provides support to my next case - it's most likely true that the introduction of multiple bounded contexts and/or subdomains will lead to more moving parts. But does that actually equate to complexity?

We've all seen over-engineered software that throws in a message queue here, another message queue over there, neither appearing to offer any discernible value. I'm certainly not advocating that! But these over imaginative solutions shouldn't be confused with DDD influenced design decisions to separate contexts and integrate them effectively where necessary.

Applying the KISS principle to architecture doesn't necessarily mean a system with fewer moving parts. A simple system is one that is highly adaptable and reactive to business changes - thus, the number of moving parts alone can't be considered a good measure of complexity.

I hope the arguments I've made in this article help to address some common misconceptions. I do believe the concept of a simple, single, all-knowing domain model is a fallacy. Feel confident to apply DDD principles, be proud of your separate cohesive models, and don't fear the additional moving parts you might adopt in the process.

Complexity is not always what it seems.












