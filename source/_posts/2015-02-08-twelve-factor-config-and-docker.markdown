---
layout: post
title: "Twelve-factor config and Docker"
date: 2015-02-08 08:03
comments: true
categories: [Docker, Twelve-factor, Configuration]
---
Config confusion
----------------

I recently wrote about what I see to be confusion over the way the [Twelve-factor app](http://www.12factor.net) guidelines are interpreted with regard to app config. You can read that post [here](http://www.dreweaster.com/blog/2015/01/21/a-confusing-side-to-Twelve-factor-app-configuration/).

To summarise my argument, I think people tend to focus solely on the explicit guidelines in the [Config](http://12factor.net/config) section, and overlook the additional advice - specific to config - given in the [Build, release, run](http://12factor.net/build-release-run) section. The former simply speaks about reading app config from the environment, the latter clearly states that a software 'Release' is a _uniquely versioned_ combination of a 'Build' _and_ 'Config'.

Suffice to say, I'm quite surprised at the extent to which people manage to overlook the [Build, release, run](http://12factor.net/build-release-run) section when discussing Twelve-factor config. It offers extremely specific advice with regard to managing _immutable_ release packages, and I don't believe it's correct to claim you're doing Twelve-factor style config if you're not also following the [Build, release, run](http://12factor.net/build-release-run) guidelines.  

In this post, I want to address the implications of this view with regard to shipping applications in Docker containers. Once again, I see some conflict in what Twelve-factor has to say about config and  perceived best practices for Docker.

The Docker way
--------------

I've digested a whole bunch of various opinions and best practices with regard to Docker, and a fairly consistent view is that containers should remain environment agnostic - i.e. the same container you generate at 'Build' time should be deployable to _any_ environment.

I get this, and I'm in total agreement. There's certainly agreement here with Twelve-factor, at least in terms of what constitues a 'Build'.  So, how would we supposedly do Twelve-factor config with this model? It appears to be quite simple as Docker lets us pass in environment variables when running a container, e.g.

`docker run -e FOO=bar coolcompany/coolapp:1.0.0`

This is saying, run `coolapp` with tag `1.0.0` passing in `bar` as the value for environment variable `FOO`. The Docker tag, in this fictitious example, is meant to represent the 'Build' version of the app, and would have been generated during the build phase in the delivery pipeline.

This approach is absolutely consistent with the Twelve-factor [Config](http://12factor.net/config) section - our application (encapsulated in the container) will read its configuration from the environment variable(s) provided. And, of course, we haven't tied the container image to a specific environment - this container looks very much like what Twelve-factor refers to as a 'Build'.

Hold on, though. Whilst we've satisfied the [Config](http://12factor.net/config) section, we've only partly satisfied the [Build, release, run](http://12factor.net/build-release-run) section. In fact, I'd go as far as saying that this is violating the [Build, release, run](http://12factor.net/build-release-run) guidelines.

Let's take some quotes directly from the Twelve-factor guidelines:

>The release stage takes the build produced by the build stage and combines it with the deploy’s current config. The resulting release contains both the build and the config and is ready for immediate execution in the execution environment.

and:

> Every release should always have a unique release ID, such as a timestamp of the release (such as 2011-04-06-20:32:17) or an incrementing number (such as v100). Releases are an append-only ledger and a release cannot be mutated once it is created. Any change must create a new release.

In our example above, I think it's fair to say that this advice has been circumnavigated. We've taken our 'Build' and jumped straight to 'Run', altogether ignoring what Twelve-factor refers to as a 'Release'. We've _not_ created a uniquely versioned, immutable release package and we've burdened the 'Run' phase with the additional responsibility of having to pass environment variables to the container. The 'Run' phase has become more complicated than it should be.

This approach has maintained a distinct separation between code and config, whereas Twelve-factor very explicitly specifies that a 'Release' _is_ a combination of code and config. The Twelve-factor approach allows the 'Run' phase to be dumb - it just launches whatever package you give it, needing no knowledge of application specific configuration. And, it naturally follows that rollbacks are a simple case of running the previously versioned release, with no need to worry about what the configuration for that version should be.

An alternative approach
-----------------------

This is where this post is bound to get murky and upset a few people. I'm going to be heretical and suggest a model whereby we do create environment specific Docker containers. I can hear the cries of "How very dare he?!"

I propose the idea of taking our base 'Build' image and creating a uniquely versioned 'Release' image as a thin layer on top of it - oh the joys of image layering. This new image _does_ embed the environment variables - specific to a chosen environment - within itself, rather than requiring they be passed to `docker run` on launch.

Let's look at an example Dockerfile to achieve this:

```
FROM coolcompany/coolapp:1.0.0
ENV FOO bar
```

We can then use this Dockerfile to build the 'Release' image, giving it a unique version at the same time, e.g. `coolcompany/coolapp:1.0.0-staging-v11`.

I've made up a convention here `{build_version}-{environment_name}-{release_number}` for tagging releases. Including the environment name in the tag might be a nice way of ensuring it's clear which environment the container is tied to.

So, our delivery pipeline continues to produce an environment agnostic container 'Build' image, but, just at the point of deployment to our chosen environment, we create a new environment specific image and use this as our 'Release'. Then, the 'Run' phase need only be given the 'Release' image version in order to execute the application.

This model sees 'Release' packages created on demand - i.e. a 'Release' package (Docker image) is create _just in time_ at the point of deployment to a specified environment. From where environment variables are actually sourced and added to the Dockerfile, is beyond the realms of this post.

The right way?
--------------

I've read enough so called best practices to expect this approach to anger some Docker/containerization purists. However, I genuinely see this as being a reasonable way to implement the Twelve-factor guidelines using Docker.

If not this approach, then what? For me, one reasonable way to challenge this model would be to challenge the whole Twelve-factor concept of [Build, release, run](http://12factor.net/build-release-run). If you disagree with the Twelve-factor concept of a 'Release', then by all means disagree with the content of this post!

Just like with my [related post](http://www.dreweaster.com/blog/2015/01/21/a-confusing-side-to-Twelve-factor-app-configuration/) - and despite being sympathetic to the [Build, release, run](http://12factor.net/build-release-run) advice - I'm not necessarily arguing right or wrong here. It's just a case of pointing out what would constitute a pure implementation of the Twelve-factor guidelines on top of Docker.

Remember, the Twelve-factor guidelines were essentially invented by the Heroku gurus, and there are other PaaS technologies that also follow the same principles. It's just a specific way of tackling release management, and, whilst it may not be the _right_ way of using Docker, I don't think it would be fair to say it's _wrong_ either.

What do you think?
