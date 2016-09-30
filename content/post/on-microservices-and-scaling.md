+++
date = "2016-09-30T12:06:01+08:00"
title = "Summary of UBER lessons on scaling microservices"
description = "TL;DW of great UBER talk on scaling microservices"
tags = [ "scaling", "microservices", "uber"]
topics = [ "scaling", "microservices", "uber"]
slug = "on-microservices-and-scaling"
+++

I just watched that amazing videos from Matt Ranney

<iframe width="560" height="315" src="https://www.youtube.com/embed/kb-m2fasdDY" frameborder="0" allowfullscreen></iframe>

Here are my takeaways and opinionated summary:

* Use RPC for service to service communications: [gRPC](http://www.grpc.io/) seems to be a good way of tackling it
* Use many repositories
* Profiling should be unified: flamegraph seems to be universal (Go profiling is great too)
* Premature optimization is bad but performance monitoring is crucial!
* Trace requests, keep context within all logs
* Log a lot, but only on a portion of your production architecture as logging can have a big cost
* Do load test in production all the time
* Systematically shut down services randomly (chaos monkey like)
* Use available aas (as a service) as much as possible
* Politics definition: when the property Company > Team > Self is violated
* Everything is a tradeoffs