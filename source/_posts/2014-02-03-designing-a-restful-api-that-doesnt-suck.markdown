---
layout: post
title: "Designing A RESTful API That Doesn't Suck"
date: 2014-02-03 14:23:30 -0500
comments: true
categories: 
---
## [Designing A RESTful API That Doesn't Suck](/blog/2013/03/22/designing-a-restful-api-that-doesn-t-suck.html)

As we're getting closer to shipping the first version of [devo.ps](http://devo.ps) and we are joined by a few new team members, the team took the time to review the few principles we followed when designing our RESTful JSON API. A lot of these can be found on [apigee's blog](https://blog.apigee.com/taglist/rest_api_design) (a recommended read). Let me give you the gist of it:

  * **Design your API for developers first**, they are the main users. In that respect, simplicity and intuitivity matter.

  * **Use [HTTP verbs](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)** instead of relying on parameters (e.g. `?action=create`). HTTP verbs map nicely with [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete):

    * `POST` for _create_,
    * `GET` for _read_,
    * `DELETE` for _remove_,
    * `PUT` for _update_ (and `PATCH` too).
  * **Use [HTTP status codes](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)**, especially for errors (authentication required, error on the server side, incorrect parameters)… There are plenty to choose from, here are a few:

    * `200`: _OK_
    * `201`: _Created_
    * `304`: _Not Modified_
    * `400`: _Bad Request_
    * `401`: _Unauthorized_
    * `403`: _Forbidden_
    * `404`: _Not Found_
    * `500`: _Internal Server Error_
  * **Simple URLs for resources: first a noun for the collection, then the item**. For example `/emails` and `/emails/1234`; the former gives you the collection of emails, the second one a specific one identified by its internal id.

  * **Use verbs for special actions**. For example, `/search?q=my+keywords`.

  * **Keep errors simple but verbose (and use HTTP codes)**. We only send something like `{ message: "Something terribly wrong happened" }` with the proper status code (e.g. `401` if the call requires authentication) and log more verbose information (origin, error code…) in the backend for debugging and monitoring.

Relying on HTTP status codes and verbs should already help you keep your API calls and responses lean enough. Less crucial, but still useful:

  * **JSON first**, then extend to other formats if needed and if time permits.
  * **[Unix time](http://en.wikipedia.org/wiki/Unix_time)**, or you'll have a bad time.
  * **Prepend your URLs with the API version**, like `/v1/emails/1234`.
  * **Lowercase everywhere in URLs**.