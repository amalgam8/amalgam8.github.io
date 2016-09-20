---
layout: page
title: Service Registry
permalink: /docs/registry/
order: 4
---

To integrate and use a microservice in Amalgam8, it needs to be registered
in the Amalgam8 Registry.  The most basic way of registering, which can be
used for microservices running on any platform / runtime, is shown in the
following diagram:

![basic registration](/docs/figures/amalgam8-service-registration.svg)

The Amalgam8 sidecar is used to register and continuously send heartbeats
to the registry on behalf of a microservice instance. In addition, the
sidecar keeps track of the health of the application (microservice A in the
figure above) and stops sending heartbeats if or when it terminates, soon
after which the registration in the Amalgam8 Registry will expire and be
removed.  Note that if the sidecar is unable to send heartbeats to the
Registry for some reason (say network disruption), the service's entry in
the Registry might expire, causing the instance information to be removed
from the Registry. In other words, the instance will become unavailable to
other microservices. When the sidecar re-establishes connectivity with the
Registry, it re-registers the service.  This is a common service
registration design pattern used in many microservice frameworks, and will
work for services running anywhere.

## Interoperability <a id="interoperability"></a>

A key design feature of Amalgam8 Registry is to leverage features of any
particular runtime platform that a microservice may be deployed in.  The
Registry supports the concept of adapters -- pluggable modules that can be
used to populate the Registry with information from other sources instead
of the sidecar, such as Etcd, Consul, Kubernetes' internal registry,
etc. Currently, the Registry provides an adapter for Kubernetes, and can be
used as a drop-in replacement for Netflix Eureka, with the added feature of
multi-tenancy.
