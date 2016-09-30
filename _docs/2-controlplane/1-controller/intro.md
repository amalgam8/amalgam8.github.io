---
layout: page
title: Introduction
permalink: /docs/control-plane/controller/
category: Control Plane
subcategory: Route Controller
order: 0
---

One, if not the most, valuable feature of Amalgam8 is its ability to
dynamically program rules for routing and manipulating requests across microservices in a running application.
The controller provides an API to configure rules for request routing, fault injection, etc.,
enabling a host of higher level functions such as
canary and red/black deployments, A/B testing, and systematically testing resilience of microservices.

The rest of this guide is organized as follows:

* [A8 Rules API](/docs/control-plane/controller/rules-api/) section describes
  the API used to manage routing and other action rules.
  
* [A8 Rules DSL](/docs/control-plane/controller/rules-dsl/) section describes
  the Amalgam8 DSL (Domain Specific Language) used to represent rules in the API.
  
* [Configuration](/docs/control-plane/controller/controller-configuration-options/) section provides
  details on running and configuration of the controller service in the control-plane.
