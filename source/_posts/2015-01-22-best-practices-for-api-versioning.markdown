---
layout: post
title: "Best practices for API versioning"
date: 2015-01-22 19:53
comments: true
categories: web
---
This is a good and a tricky question. The topic of URI design is at the same time the most prominent part of a REST API and, therefore, a potentially long-term commitment towards the users of that API.

Since evolution of an application and, to a lesser extent, its API is a fact of life and that it's even similar to the evolution of a seemingly complex product like a programming language, the URI design should have less natural constraints and it should be preserved over time. The longer the application's and API's lifespan, the greater the commitment to the users of the application and API.

On the other hand, another fact of life is that it is hard to foresee all the resources and their aspects that would be consumed through the API. Luckily, it is not necessary to design the entire API which will be used until [Apocalypse](http://en.wikipedia.org/wiki/Apocalypse). It is sufficient to correctly define all the resource end-points and the addressing scheme of every resource and resource instance.

Over time you may need to add new resources and new attributes to each particular resource, but the method that API users follow to access a particular resources should not change once a resource addressing scheme becomes public and therefore final.

This method applies to HTTP verb semantics (e.g. PUT should always update/replace) and HTTP status codes that are supported in earlier API versions (they should continue to work so that API clients that have worked without human intervention should be able to continue to work like that).

Furthermore, since embedding of API version into the URI would disrupt the concept of [hypermedia as the engine of application state](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_5) (stated in Roy T. Fieldings PhD dissertation) by having a resource address/URI that would change over time, I would conclude that API versions should not be kept in resource URIs for a long time meaning that <font color="green">resource URIs that API users can depend on should be permalinks</font>.


Sure, it is possible to embed API version in base URI but only for reasonable and restricted uses like debugging a API client that works with the the new API version. Such versioned APIs should be time-limited and available to limited groups of API users (like during closed betas) only. Otherwise, you commit yourself where you shouldn't.

A couple of thoughts regarding maintenance of API versions that have expiration date on them. All programming platforms/languages commonly used to implement web services (Java, .NET, PHP, Perl, Rails, etc.) allow easy binding of web service end-point(s) to a base URI. This way it's easy to gather and keep a collection of files/classes/methods separate across different API versions. 

From the API users POV, it's also easier to work with and bind to a particular API version when it's this obvious but only for limited time, i.e. during development.

From the API maintainer's POV, it's easier to maintain different API versions in parallel by using source control systems that predominantly work on files as the smallest unit of (source code) versioning.

However, with API versions clearly visible in URI there's a caveat: one might also object this approach since <font color="red">API history becomes visible/aparent in the URI design</font> and therefore is prone to changes over time which goes against the guidelines of REST. I agree!

The way to go around this reasonable objection, is to implement the latest API version under versionless API base URI. In this case, API client developers can choose to either:

  - develop against the latest one (committing themselves to maintain the application protecting it from eventual API changes that might break their badly designed API client).

  - bind to a specific version of the API (which becomes apparent) but only for a limited time

For example, if API v3.0 is the latest API version, the following two should be aliases (i.e. behave identically to all API requests):
```
http://shonzilla/api/customers/1234
http://shonzilla/api/v3.0/customers/1234
http://shonzilla/api/v3/customers/1234
```

In addition, API clients that still try to point to the *old* API should be informed to use the latest previous API version, if the API version they're using is obsolete or not supported anymore. So accessing any of the obsolete URIs like these:
```
http://shonzilla/api/v2.2/customers/1234
http://shonzilla/api/v2.0/customers/1234
http://shonzilla/api/v2/customers/1234
http://shonzilla/api/v1.1/customers/1234
http://shonzilla/api/v1/customers/1234
```
should return any of the 30x HTTP status codes that indicate redirection that are used in conjunction with `Location` HTTP header that redirects to the appropriate version of resource URI which remain to be this one:
```
http://shonzilla/api/customers/1234
```
There are at least two redirection HTTP status codes that are appropriate for API versioning scenarios:

  - [301 Moved permanently](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3.2) indicating that the resource with a requested URI is moved permanently to another URI (which should be a resource instance permalink that does not contain API version info). This status code can be used to indicate an obsolete/unsupported API version, informing API client that a versioned resource URI been replaced by a resource permalink.

  - [302 Found](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3.3) indicating that the requested resource temporarily is located at another location, while requested URI may still supported. This status code may be useful when the version-less URIs are temporarily unavailable and that a request should be repeated using the redirection address (e.g. pointing to the URI with APi version embedded) and we want to tell clients to keep using it (i.e. the permalinks).

  - other scenarios can be found in [Redirection 3xx chapter of HTTP 1.1 specification](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3)