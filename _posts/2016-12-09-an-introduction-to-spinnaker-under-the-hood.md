---
title: "An Introduction To Spinnaker: Under the Hood"
date:   2016-12-09
redirect_from: /2016/12/08/an-introduction-to-spinnaker-under-the-hood/
---

I've been a part of the [Spinnaker](http://spinnaker.io/)  team for about 18
months now. If you're unfamiliar, Spinnaker is a tool for doing safe and
continuous deployment  to a cloud environment. Check out the link above for much
more detail about what it is and what you can do with it - I won't rehash that
here.

In that time, I've had the pleasure of doing a lot of refactors of the
components to make the product multi-cloud, which at the time meant Amazon Web
Services (AWS) and the Google Compute Engine (GCE).

In this series of blog posts, I plan on doing deep dives into several of the
components in order to help others interested in knowing how Spinnaker works
under the hood, and some of the development decisions that went into building
it.

First, we'll take the 10,000 foot view. Spinnaker is composed of several
micro-services:


This is a non-exhaustive list of the main components:

* Deck  - UI console. For production instances, HTML, CSS, and JS files are
   generated statically and served up through a standard Apache server. In
   development, we use a local Node server.
* Gate  - API Gateway. I revamped the authentication module here. (More on that
   in the future).
* Orca  - Orchestration engine. Pipelines and ad-hoc UI operations all run
   through here.
* Front50  - Front end for persistently stored configuration files. Things like
   Pipelines and Application configurations are "owned" by Front50. * History:
      Used to store things in Cassandra, but this storage is deprecated. We now
      encourage users to use a cloud platform-provided mechanism (i.e. storage
      buckets like S3 or Google Cloud Storage).


* Clouddriver  - Cloud platform interface. Caches current view of all resources
   that actually live in the cloud. Requests for changes (mutations) all go
   through Clouddriver. * History: Used to be 3 separate components: "kato"
      (write), "oort" (read), and "mort" (infrastructure read).


 * Rosco  - Baker of cloud VM images. Uses [Packer](https://www.packer.io/)
   behind the scenes to do the heavy lifting.

   Each of these micro-services is a [Spring Boot](https://projects.spring.io/spring-boot/)  based application coded mostly in
Groovy (except for Deck, which is a Node based Javascript single page
application).

The team is pushing for all new components to be in Java 8, since it has finally
caught up to some of the niceties Groovy has, with stronger typing.

Except for collection literals - I really wish Java 8 had those.

```groovy
def myList = ["item1"]
def myMap = ["key": value]
```

is much nicer than:

```java
List<String> myList = new ArrayList<>();
myList.add("item1");

Map<String, Object> myMap = new HashMap<>();
myMap.put("key", value);
```

I digress...

Each of the backend components also follows the scalability design pattern of
separating out state from execution. I'll dive more into this in a later post,
because I think it's an important software architectural design pattern.
