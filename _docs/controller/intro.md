---
layout: page
title: Introduction
permalink: /docs/controller/
category: Controller
order: 0
---

The Amalgam8 Route Controller is a multi-tenant, highly-available service
for managing routing across microservices via the
[sidecar](/docs/sidecar/). Routes programmed at the controller are percolated
down to the sidecars periodically. Routing rules can be based on the
content of the requests and the version of microservices sending and
receiving the requests. In addition to routing, rules can also be expressed
for injecting faults into microservice API calls.

**High Availability:** The Route Controller supports a Redis backend for
storing routing information. In this mode, the Controller instances are
stateless and can be scaled using standard autoscaling techniques. In
tandem, the data store backend, i.e., Redis, needs to be deployed in HA
mode. Please refer to the
[Redis Clustering](http://redis.io/topics/cluster-tutorial) documentation
for details on setting up a highly available Redis backend.

[Authentication & Multi-tenancy](/docs/controller/controller-authentication/)
section describes running the controller in single and multi-tenant modes
and the forms of authentication available.
