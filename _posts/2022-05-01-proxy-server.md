---
layout: post
title: "TIL: Reverse Proxy Servers"
description: "TIL: Reverse Proxy Servers"
date: 2022-01-05
tags: [system-design]
---

Today I learnt about reverse proxy servers.

<br> 

Reverse proxy server is a server that takes a client request and forwards it to the backend server. So basically it's in between of the client and origin server itself. The difference between Content Delivery Network and reverse proxies is caching - a CDN reverse proxy caches responses from the origin servedr that are on their way back to the client. 

Benefits of using reverse proxy servers:
* security, 
* SSL,
* scalability.

About scalability - if you have a direct communication between clients and origin servers, it might happen that a single web server can't handle all the traffic or gets attacked, thus there's proxy server in between.

Example: 
* Apple Trailers uses Akamai
* JQuery.com hosts its JavScript files using CloudFront CDN