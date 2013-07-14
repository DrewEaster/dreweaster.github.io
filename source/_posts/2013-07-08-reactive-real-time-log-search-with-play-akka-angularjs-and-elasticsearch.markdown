---
layout: post
title: "Reactive, real-time log search with Play, Akka, AngularJS and Elasticsearch"
date: 2013-07-08 12:25
comments: true
categories: [Play, Scala, Akka, AngularJS, Elasticsearch]
---
Introduction
------------

So, I've decided to contribute an Activator Template to TypeSafe (will submit soon, promise!). Having recently become more and more involved in Elasticsearch, I saw a great opportunity to put together a neat "reactive" application combining Play &amp; Akka with the "bonsai cool" [percolation](http://www.elasticsearch.org/guide/reference/api/percolate/) feature of Elasticsearch. Then, to put a cherry on top, use AngularJS on the client-side to create a dynamically updating UI.

What I came up with is slightly contrived - a very basic real-time log entry search tool - but I think it provides a really nice base for apps that want to integrate this bunch of technologies.

{% img /images/real-time-search.png/600/264 %}

All the code for the application is available on Github [here](https://github.com/drewzilla/realtime-search). In this post, I've attempted to dissect the Activator Template tutorial I've written and regurgitate it in blog form.

### Play and Akka

Play and Akka are used to implement the reactive server-side application. The application favours SSE (Server Sent Events) to push updates to the client. The template introduces a number of interesting topics, including Play Iteratees/Enumerators and Akka Actors.

### AngularJS

AngularJS has been chosen on the client-side to demonstrate how simple it can be to build a dynamic, single page user experience with very little code.

### Elasticsearch

The "bonsai cool" percolation feature of Elasticsearch achieves the real-time search aspect of the application. The application starts up an embedded Elasticsearch node, so no need to run your own external instance. Take a look at [EmbeddedESServer](https://github.com/drewzilla/realtime-search/blob/master/app/utils/EmbeddedESServer.scala) for the embedded server code. There is a custom [Global](https://github.com/drewzilla/realtime-search/blob/master/app/Global.scala) where the embedded server is started and shutdown as part of the application lifecycle.

The Actors
----------

The application has three actors:

    - [MainSearchActor](https://github.com/drewzilla/realtime-search/blob/master/app/actors/MainSearchActor.scala) supervises and coordinates data flows
    - [ElasticsearchActor](https://github.com/drewzilla/realtime-search/blob/master/app/actors/ElasticSearchActor.scala) interacts with Elasticsearch percolation API
    - [LogEntryProducerActor](https://github.com/drewzilla/realtime-search/blob/master/app/actors/LogEntryProducerActor.scala) generates 'random' log entry data

### MainSearchActor

This actor's job is to coordinate the reactive parts of the application and supervise the other actors. It is the main dependency of the application's single Play [controller](https://github.com/drewzilla/realtime-search/blob/master/app/controllers/Application.scala).

__Starting/stopping a search__

The actor responds to a `StartSearch` message by 'replying' with an `Enumerator` to the sender. The `Enumerator` wraps a unicast channel to which log entries are pushed that match the query string sent within the message. Let's take a look at some code:

{% codeblock lang:scala %}
private def startSearching(startSearch: StartSearch) =
    Concurrent.unicast[JsValue](
        onStart = (c) => {
            channels += (startSearch.id -> c)
            elasticSearchActor ! startSearch
        },
        onComplete = {
            self ! StopSearch(startSearch.id)
        },
        onError = (str, in) => {
            self ! StopSearch(startSearch.id)
        }
    ).onDoneEnumerating(
        callback = {
            self ! StopSearch(startSearch.id)
        }
    )
{% endcodeblock %}

The Play Iteratees library has the very handy `Concurrent` utilities. In this case, `Concurrent.unicast` is called to create an `Enumerator` that encloses a `Concurrent.Channel`. When the channel starts (`onStart`), it is stored in a map local to the actor (using UUID as key) and the `StartSearch` message is forwarded onto the `ElasticSearchActor` where the query will be percolated in Elasticsearch. It's worth noting that this code is not production ready - it ought to be a transactional operation, i.e. we should only store the channel once we know Elasticsearch has successfully percolated the query. You will notice that a `StopSearch` message is sent to `self` such that the channel is removed from the local map, and the percolated query is deleted, when the channel is no longer useful (i.e. is closed by the client, or an error occurs).

__Broadcasting matching results__

The actor will receive a `SearchMatch` message when a log entry has matched a percolated query.

{% codeblock lang:scala %}
private def broadcastMatch(searchMatch: SearchMatch) {
    searchMatch.matchingChannelIds.foreach {
        channels.get(_).map {
            _ push searchMatch.logEntry.data
        }
    }
}
{% endcodeblock %}

On receipt of the message, each matching id is iterated over and the corresponding channel is retrieved from the local map. The log entry is then pushed to the channel, and thus onto the client.

__Scheduling log entry creation__

The actor uses the Akka scheduler to send a `Tick` to the `LogEntryProducerActor` every second - in the real world, this would obviously be unnecessary, as genuine log entries would be fed into the application in some other way. The `Tick` is sent to `self` before being forwarded on to the `LogEntryProducerActor`.

### ElasticsearchActor

This actor has responsibility for both registering queries in Elasticsearch and percolating log entry documents against those queries. Rather than utilise the Elasticsearch Java Client, the code, instead, crafts the Elasticsearch API calls manually, demonstrating the use of the asynchronous Play WS API to execute them. For simplicity, the calls are hard coded to talk to Elasticsearch on localhost:9200 (where the embedded server will be listening).

The code is fairly self explanatory within this actor. Do note that there is a lack of error handling on the API calls thus making this actor unsuitable for production use in its current form. It is recommended you read the Elasticsearch [documentation on percolation](http://www.elasticsearch.org/guide/reference/api/percolate/) to learn more about this powerful feature.

There's one little important gotcha this code has avoided - closing over the `sender` ref in an asynchronous callback block. The `sender` ref is part of the shared mutable state of the actor and so, if the actor were to reply to the sender in the percolate callback, a race condition would be encountered if another thread had modified the actor's state before the percolation call to Elasticsearch had completed. This race condition has been avoided by ensuring to 'freeze' the `sender` ref, by sending it to a private function:

{% codeblock lang:scala %}
private def percolate(logJson: JsValue, requestor: ActorRef)
{% endcodeblock %}

and close over the parameter instead.

### LogEntryProducerActor

I won't go into the detail of this actor. Suffice to say, its job is to generate a random, JSON formatted log event whenever it receives a `Tick` message. In reality, a genuine source of log events would replace this actor.

The Play Controller
-------------------

As most of the server-side logic exists within the actors, the single Play controller is very simple. The most interesting aspect of the controller is the action that opens an event stream connected with the client:

{% codeblock lang:scala %}
def search(searchString: String) = Action {
    Async {
        (searchActor ? StartSearch(searchString = searchString)).map {
            case SearchFeed(out) => Ok.stream(out &> EventSource()).as("text/event-stream")
        }
    }
}
{% endcodeblock %}

The most important thing to note is the use of the Akka 'ask' pattern of message exchange (notice the use of '?' instead of '!'). This differs from the more typical fire-and-forget approach in that we're able to asynchronously pick up a reply from the recipient actor. In this scenario, a <code>StartSearch</code> message is sent to the <code>MainSearchActor</code> which replies with an <code>Enumerator</code> used to stream search results to the client. Given the use of the 'ask' pattern, we wrap the action logic in an <code>Async</code> block - so not to hold up other requests - rather than blocking until the <code>Future</code> yields a result.

The User Interface
------------------

The key parts of the application UI are:

    1. A Play [template](https://github.com/drewzilla/realtime-search/blob/master/app/views/index.scala.html) with AngularJS specific markup
    2. A single AngularJS [controller](https://github.com/drewzilla/realtime-search/blob/master/app/assets/javascripts/controllers.js)

The application makes use of the [WebJars](http://www.webjars.org/) project to simplify the introduction of its JS and CSS dependencies (e.g. AngularJS and Twitter Bootstrap).

### UI Template

Firstly, the opening `<div>` is linked to the controller `SearchCtrl` that subsequently enables the automagical databinding power of AngularJS. A simple search form captures an Apache Lucene formatted query string. A search can be started by clicking on the 'Search' button which invokes the `startSearching()` function defined in the controller. Finally, you can see the use of AngularJS two-way databinding to render matching search results contained within the view model (only displays the latest 10 matches):

{% codeblock lang:scala %}
<tr ng-repeat="searchResult in searchResults | limitTo:10">
{% endcodeblock %}

### The AngularJS Controller

The AngularJS controller is fairly straightforward. The key part is handling a new search:

{% codeblock lang:scala %}
$scope.startSearching = function () {
    $scope.stopSearching()
    $scope.searchResults = [];
    $scope.searchFeed = new EventSource("/search/" + $scope.searchString);
    $scope.searchFeed.addEventListener("message", $scope.addSearchResult, false);
};
{% endcodeblock %}

Firstly, an existing search is stopped (if running) and the results model cleared (will automagically clear any existing results from the HTML markup). Secondly, an event stream connection is made to the server and an event listener is added that pushes matching search results into the model as they arrive from the server.

As we are updating the model (in the `addSearchResult` function) outside of it's knowledge, we've had to explicitly tell Angular that we've pushed a new result to the model by wrapping the code in a function passed to `$scope.apply`. See the Angular docs for the [$scope](http://docs.angularjs.org/api/ng.$rootScope.Scope) object for more information.

Using the application
---------------------

Fire up the application using `play run` and point your browser (not IE - it doesn't support SSE. Doh!) to [http://localhost:9000](http://localhost:9000). Using the application is as simple as entering an Apache Lucene formatted query. Please read the Lucene [query parser syntax documentation](http://lucene.apache.org/core/4_3_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html#package_description) for more information on how to construct Lucene query strings. A simple example would be to enter "GET" as your query string, thus matching all log entries for GET requests.

Enjoy!