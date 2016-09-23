---
layout: page
title: Introduction
permalink: /docs/sidecar/
category: Sidecar
order: 0
---

The Almagam8 sidecar has a flexible architecture that can be configured and
used by applications in a number of ways. The sidecar can be split into the
following functional components:

![sidecar architecture](/docs/figures/amalgam8-sidecar-components.svg)

* **Service Registration** - Registers a microservice instance in the Registry and sends periodic heartbeats.
* **Route Management** - Receives route updates from the Controller and updates the Proxy without disruption.
* **Supervisor** - Manages the lifecycle of an Amalgam8 Proxy server and optionally the associated microservice itself.
* **Proxy** - An [OpenResty-based Nginx](https://openresty.org/en/) server  that acts as a request forwarder and load-balancer.

The rest of this guide is organized as follows:

* [Deployment Options](/docs/sidecar/deployment-options/) section describes
  different ways of packaging the sidecar with a microservice (as part of a
  container or a pod)
  
* [Usage](/docs/sidecar/usage/) section describes how to use the
  sidecar as a proxy to make API calls to dependent microservices.
  
* [Configuration](/docs/sidecar/sidecar-configuration-options/) section provides details on
  various runtime configuration options while launching the sidecar.

    * [Automatic Service Registration](/docs/sidecar/sidecar-auto-service-registration/)
      describes how to setup the sidecar to automatically register the
      microservice with the Amalgam8 Service Registry.

    * [Intelligent Request Routing](/docs/sidecar/sidecar-request-routing/)
      describes how to setup the sidecar for request proxying wherein it
      performs content and version-based routing routing based on the rules
      provided by the control plane.

    * [Logging](/docs/sidecar/sidecar-logging-api-calls/)
      describes how the sidecar logs the API calls and which log files need
      to be managed by the user.
