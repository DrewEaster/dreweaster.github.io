---
layout: post
title: "Applying the builder pattern in Scala"
date: 2015-01-27 22:05
comments: true
categories: [Scala, Design Patterns]
published: false
---

If you're an object-oriented programmer, you're unlikely to have avoided the telescoping constructor anti-pattern on your travels. This rather funky sounding anti-pattern is the result of allowing for a number of variations for object construction, leading to an exponential growth in constructors.

[telescoping constructor anti-pattern example]

This pain is particularly noticeable when subsequently introducing additional construction parameters into the mix - you're then faced with some very fiddly decisions as to what new constructors you need, and what existing ones might need to be modified.

For simple cases, Scala provides out-of-the-box solutions that avoid the telescoping constructor anti-pattern - the ability to define default values to constructor parameters is your friend here.

[Default values code example]

Combined with the ability to name parameters as you define them, it's pretty clear you can get close to applying the Builder Pattern in Scala without actually needing to explicitly define a separate companion builder for a class.

[Default values with named parameters code example]

There are some cases where the native language features are not quite what you want. It's not uncommon for a Builder to encapsulate some specific logic associated with it's companion class's construction. This might be done to keep interactions with the Builder as simple as possible, avoiding the need for the caller to understand some of the nuances of the underlying construction mechanism. For example:

[Code example with dates?]
