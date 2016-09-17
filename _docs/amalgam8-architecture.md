---
layout: page
title: Amalgam8 Architecture
permalink: /docs/architecture/
order: 1
---

The Amalgam8 platform consists of the following three components:

1. **Registry** - A high performance multi-tenant service registry that
   provides a centralized view of all the microservices in an application,
   regardless of where they are actually running. To simplify integration
   of microservices that are already using a service registry, Amalgam8
   registry provides a plug-in point for adapters that can automatically
   pull in information from
   other service registry solutions such as Etcd (used by Kubernetes),
   Netflix Eureka, and Spring Cloud Discovery.

2. **Controller** - A multi-tenant service for managing routing across
   microservices via the sidecars. Routes programmed at the controller are
   percolated down to the sidecars periodically. Routing rules can be based
   on the content of the requests and the version of microservices sending
   and receiving the requests. In addition to routing, rules can also be
   expressed for injecting faults into microservice API calls.

3. **Sidecar** - Each microservice in the application is paired with a
   sidecar. Conceptually, the sidecar's basic responsibility includes
   service registration (where required), service discovery and
   *client-side load-balancing* of requests across microservice
   instances. In addition to these basic responsibilities, the sidecar is
   responsible for essential functions such as content and version-based
   routing, fault injection, ACLs, rate limiting, etc. From a design
   standpoint, the choice of technology for the sidecar is irrelevant. *Any
   implementation of the sidecar is sufficient (as a library or a helper
   process) as long as the sidecar performs the above functions and is
   dynamically programmable at runtime*.
   
   Amalgam8's sidecar implementation uses OpenResty + Nginx for request
   routing, and employs a simple daemon for receiving and updating the routing
   rules in Nginx, and for service registration & heartbeat (where needed).

Applications run as tenants of the Registry and the Controller. They
register their services in the Registry and use the Controller to manage
the sidecars. The following diagram illustrates how the three components
work together.

![how it works](/docs/figures/how-amalgam8-works.svg)

1. Microservice instances are registered in the service registry by the
   sidecars. There are several ways this may be accomplished as described in
   [Amalgam8 Registry](#service-registration).
2. The Developer uses the control plane API to configure high-level rules
   for request routing between services (e.g., splitting traffic across
   versions, injecting delays).
3. The route controller translates these rules into low-level control information
   and sends them to the sidecars.
4. A microservice invokes APIs of other microservices by pointing to the
   sidecar as the destination host. For e.g., [http://localhost:6379/serviceName/apiEndpoint]()
5. The sidecar (which is based on Nginx/OpenResty) forwards the request to the
   appropriate microservice, depending on the request path and routing
   rules specified by the controller.
