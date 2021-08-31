---
title: HTTP Redirects and Funny behaviour
description: HTTP isn't as easy as you think, especially with redirects.
date: 2019-06-13
tags: ['Doh']
layout: post
---

Recently I had a weird problem reported of an application I made:

> When I submit a form it sometimes fails and I get an empty form again. But only _sometimes_.

It took quite a while to find out what actually happened. What made it hard to
debug (but finally gave me the answer) is that the posted information didn't end
up in the server. So either the browser or some middleware was messing it up.

Locally I never had any problems. It _always_ worked. But the client was using confidential information in their tests. This turned out a red herring.

The solution was to look at the requests made by the client:
whenever it went wrong, there was a 302 redirect to authenticate again. Turns out if a POST request results in a redirect, it will be a GET request, essentially all POST information gets lost [See Stack Overflow](https://stackoverflow.com/questions/33214717/why-post-redirects-to-get-and-put-redirects-to-put). After authenticating Keycloak redirected back, to an empty form. For the client, who didn't see the redirect, it was as if the server just lost their input.

For working with OIDC I noticed that the OIDC library only redirects if it cannot refresh the token via a direct HTTP call to the server. Additionally, I was using multiple workers
for the Flask application _without_ distributed storage of OIDC information.

So the solution was:

* Let people be logged in longer (internal application, so that's fine);
* Create a simple storage to share information between workers, so refreshes go via an HTTP call not a redirect.
