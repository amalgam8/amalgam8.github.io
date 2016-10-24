---
layout: page
title: Introduction
permalink: /docs/sidecar.html
category: Sidecar
order: 0
---

The Amalgam8 sidecar has a flexible architecture that can be configured and
used by applications in a number of ways. The sidecar can be split into the
following functional components:

![sidecar architecture](/docs/figures/amalgam8-sidecar-components.svg)

* **Service Registration** - Registers a microservice instance in the Registry and sends periodic heartbeats.
* **Route Management** - Receives route updates from the Controller and updates the Proxy without disruption.
* **Supervisor** - Manages the lifecycle of an Amalgam8 Proxy server and optionally the associated microservice itself.
* **Proxy** - An [OpenResty-based Nginx](https://openresty.org/en/) server  that acts as a request forwarder and load-balancer.

The rest of this guide is organized as follows:

* [Deployment Options](/docs/sidecar-deployment-options.html) section describes
  different ways of packaging the sidecar with a microservice (as part of a
  container or a pod)
  
* [Usage](/docs/sidecar-usage.html) section describes how to use the
  sidecar as a proxy to make API calls to dependent microservices.
  
* [Configuration](/docs/sidecar-configuration.html) section provides details on
  various runtime configuration options while launching the sidecar, including
  automatic service registration, health checks, and request proxying.

* [Logging](/docs/sidecar-logging-api-calls.html) describes how the
  to monitor API calls made by the sidecar.
