---
layout: page
title: Introduction
permalink: /docs/control-plane/
category: Control Plane
order: 0
---

The Amalgam8 Control Plane is the management layer that provides a cohesive
view of the microservices in the application and how they are communicating
with each other. It consists of two multi-tenant services:

1. **Route Controller** manages how requests are routed across
   microservices via the [Amalgam8 Sidecars](/docs/sidecar/). Routes
   programmed at the controller are percolated down to the sidecars
   periodically. Routing rules can be based on the content of the requests
   and the version of microservices sending and receiving the requests. In
   addition to routing, rules can also be expressed for injecting faults
   into microservice API calls.

2. **Service Registry** provides a centralized view of all the
   microservices in an application, regardless of where they are actually
   running. To simplify integration of microservices that are already using
   other service registries, Amalgam8 service registry provides a plug-in
   point for adapters that can automatically pull in information from other
   service registry solutions such as Kubernetes and Netflix Eureka.

Please refer to the following subsections for more details
on how to setup and operate the services in the control plane:

* [Route Controller](/docs/control-plane/controller/)

* [Service Registry](/docs/control-plane/registry/)
