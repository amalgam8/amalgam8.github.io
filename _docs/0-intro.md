---
layout: page
title: Introduction
permalink: /docs/introduction/
order: 0
---

Before describing Amalgam8, it is important to understand two key concepts
that are essential to any
[microservice application](http://martinfowler.com/articles/microservices.html),
namely content and version-based routing.

## Content and version-based routing - 101

In any realistic production deployment, there are typically multiple
versions of microservices running at the same time, as you might be testing
out a new version, troubleshooting an old version, or simply keeping the
old version around just in case.

*Content-based routing* allows you to route requests between microservices
based on the content of the request, such as the URL, HTTP headers,
etc. For example,

```
from microservice A, if request has "X-User-Id: QA", route to instance of
(B:v2) else route to instance of (B:v1)
```

*Version-based routing* allows you to control how different versions of
microservices can talk to each other. For example,

```
from microservice A:v2 route all requests to B:v2
from microservice A:v1 route 10% of requests to B:v2 and 90% to B:v1
```

A simple way to accomplish these functions is to control how
microservices can talk to each other.

## Amalgam8

Amalgam8 is a platform for building polyglot microservice applications that
enables you to route requests between microservices in a *content-based*
and *version-based* manner, independent of the underlying container
orchestration layer
([Docker Swarm](https://www.docker.com/products/docker-swarm),
[Kubernetes](https://kubernetes.io),
[Marathon](https://mesosphere.github.io/marathon/)) or the cloud platform
(Amazon AWS, IBM Bluemix, Google Cloud Platform, Microsoft Azure, etc.

Amalgam8 uses the sidecar model or the ambassador pattern for building
microservices applications. The sidecar runs as in independent process and
takes care of service registration, discovery and request routing to
various microservices. The sidecar model simplifies development of polyglot
applications.

Through the Amalgam8 Control Plane, you can dynamically program the
sidecars in each microservice and control how requests are routed between
microservices. The control plane provides REST APIs that serve as the basis
for building tools for various DevOps tasks such as
[A/B testing](https://www.optimizely.com/ab-testing/), internal releases
and dark launches,
[canary releases](http://martinfowler.com/bliki/CanaryRelease.html),
red/black deployments,
[resilience testing](https://developer.ibm.com/open/2016/06/06/systematically-resilience-testing-of-microservices-with-gremlin/),
etc.

![high-level architecture](/docs/figures/amalgam8-architecture.svg)
