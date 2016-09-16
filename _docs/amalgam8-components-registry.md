---
layout: page
title: Amalgam8 Registry
permalink: /docs/registry/
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

### Kubernetes-Registry Adapter <a id="k8s_adapter"></a>

A microservice running in Kubernetes with a corresponding service
definition, will already have the endpoints (instances) being tracked by
the Kubernetes runtime (kubelet) and reflected in the Kubernetes service
registry. Instead of using the sidecar to explicitly registering this
service once again in the Registry, Amalgam8 provides a Kubernetes
adapter. The adapter can be configured to watch the Kubernetes registry and
automatically mirror the endpoints in Amalgam8 as shown in the figure
below.

![k8s adapter](/docs/figures/amalgam8-registry-k8s-adapter.svg)

In the above figure, service B is a Kubernetes service defined in
serviceB.yaml. There is nothing that needs to be added or changed to
integrate it with an Amalgam8 system. Instances, as they come and go, will
be maintained in the Kubernetes registry by the Kubernetes runtime, and
then automatically mirrored in the Amalgam8 Registry by the
kubernetes-adapter.

Similar adapter plugins can be added to support other runtime environments
where service registration is automatically managed by the container
platform.

### Drop-in Replacement for Eureka <a id="eureka"></a>

Amalgam8 registry is fully API compatible with
[Netflix Eureka](https://github.com/Netflix/eureka) API version 1.1.x (later
versions may also work but have not been tested). 
As such, clients compatible with that version including, for example,
[Ribbon](https://github.com/Netflix/ribbon) and
[Spring Cloud service discovery](https://spring.io/guides/gs/service-registration-and-discovery/),
can be migrated to using Amalgam8 registry with only configuration changes.

It is important to note that while the integration is simple and requires
only minimal configuration changes, it does not enable the full range of
services in Amalgam8 such routing rules and resiliency testing. Ribbon will
not use the sidecar for client-side load balancing


#### Eureka Client Configuration (Ribbon)

Eureka service discovery is designed and used in single tenant environment,
having a shared namespace, visible to all users. Since Amalgam8 can support
multi-tenancy and multiple namespaces, some indication of the accessed
namespace is required in all calls.

Amalgam8 Service Registry encodes the namespace as a path component in the
Eureka URL configured for the client. Recent Eureka versions added
configuration options for extra headers,future Amalgam8 version may take
advantage of that to encode the namespace in the HTTP `Authorization`
header instead of the URL.

From a usage perspective, the Eureka API base URL should be changed to
include the namespace (e.g., `/api/eureka/<namespace-or-token>/v2`. Actual
changes to the Eureka client configuration depend on the target server
definition used. By default, Eureka client searches for the property file
`eureka-client.properties` in the _classpath_ (see
[here](https://github.com/Netflix/eureka/wiki/Configuring-Eureka#configuration)).

```
# Eureka base URL is: ${A8_REGISTRY_URL}/api/eureka/${A8_TOKEN}/v2/`
# Port should be changed to match the actual deployment setup

# for DNS based specification, using the 'default' namespace. 
eureka.eurekaServer.port=8080
eureka.context=api/eureka/default/v2
 
# for URL based specification (change the host and port according to your set up)
eureka.serviceUrl.default=http://<amalgam8-registry-hostname>:8080/api/eureka/default/v2/
```

The trailing slash in the serviceUrl is required and expected by the Eureka
client.

The URL based specification can be used on IBM Bluemix as well,
substituting the service credentials (i.e., the VCAP_SERVICES entry) in the
URL:

```
eureka.serviceUrl.default=${VCAP_SERVICE_URL}/api/eureka/${VCAP_AUTH_TOKEN}/v2/
```

The above changes are sufficient to enable the Eureka client to interact
with the Amalgam8 registry, and can be validated by issuing an HTTP GET to
the /apps endpoint under the base URL.

#### Spring Cloud Integration

Spring Cloud uses the Eureka client API, but it's configuration is based on
a YAML file (typically `.../src/main/resources/bootstrap.yml`). The
following code snippet shows the configuration changes:

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://<amalgam8-registry-hostname>:8080/api/eureka/default/v2/
```

Different namespaces can be used for the defaultZone by substituting
`default` with the appropriate namespace or token.
