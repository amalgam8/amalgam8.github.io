---
layout: page
title: Introduction
permalink: /docs/registry/
category: Service Registry
order: 0
---

To integrate and use a microservice in Amalgam8, it needs to be registered
in the Amalgam8 Registry.  The most basic way of registering, which can be
used for microservices running on any platform / runtime, is shown in the
following diagram:

![basic registration](/docs/figures/amalgam8-service-registration.svg)

**Registration, heartbeats and health checks**. The
[Amalgam8 Sidecar](/docs/sidecar/) is used to register and continuously
send heartbeats to the registry on behalf of a microservice instance. In
addition, the sidecar keeps track of the health of the application
(microservice A in the figure above) and stops sending heartbeats if or
when it terminates, soon after which the registration in the Amalgam8
Registry will expire and be removed.

If the sidecar is unable to send heartbeats to the Registry for some reason
(say network disruption), the service's entry in the Registry might expire
after the `TTL` period, causing the instance information to be removed from
the Registry. In other words, the instance will become unavailable to other
microservices. When the sidecar re-establishes connectivity with the
Registry, it re-registers the service.  This is a common service
registration design pattern used in many microservice frameworks, and will
work for services running anywhere.

**Interoperability**.  A key design feature of Amalgam8 Registry is to
leverage features of any particular runtime platform that a microservice
may be deployed in.  The Registry supports the concept of catalogs --
pluggable modules that can be used to populate the Registry with
information from other sources instead of the sidecar, such as Etcd,
Consul, Kubernetes' internal registry, Netflix Eureka, etc.

Currently, the Registry provides catalog adapters that can obtain service
registration information from Kubernetes, Eureka.  In addition, the
Amalgam8 Registry can be used directly as a drop-in replacement for Netflix
Eureka without the need for an explicit catalog.

**Registering external services**.  Amalgam8 is typically used to control 
communication between internal application microservices. However, routing
rules may also be injected for external cloud- and Web-based services, such
social network sites, hosted database services that are part of the cloud
platform, etc.  This extends Amalgam8's benefits, such as resiliency
testing, to include the full set of services used by the application.  To
enable specifying routing rules on external services, they must first be
visible via the Amalgam8 registry. The
[File catalog](/docs/registry/file-catalog/) can be used to register such
external services with the registry.

The rest of this guide is structured as follows:

* [Eureka API Compatibility](/docs/registry/eureka-compatibility) describes
  how to use the Amalgam8 Registry as a drop-in replacement for Netflix
  Eureka, and configure clients such as Ribbon or Spring Cloud applications
  that use the Eureka API.

* [File Catalog](/docs/registry/file-catalog/) describes how to use the
  file-based catalog to register external services such as third party
  APIs with the Amalgam8 Registry.
  
* [Kubernetes Catalog](/docs/registry/kubernetes-catalog/) describes how to
  configure the Amalgam8 Registry to synchronize registration information
  from Kubernetes' built in service registry.
