---
layout: page
title: Eureka API Compatibility
permalink: /docs/control-plane-registry-eureka-api-compatibility.html
redirect_from: /docs/control-plane/registry/eureka-compatibility/
category: Control Plane
subcategory: Service Registry
order: 2
---

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


# Eureka Client Configuration (Ribbon)

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

```properties
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

```properties
eureka.serviceUrl.default=${VCAP_SERVICE_URL}/api/eureka/${VCAP_AUTH_TOKEN}/v2/
```

The above changes are sufficient to enable the Eureka client to interact
with the Amalgam8 registry, and can be validated by issuing an HTTP GET to
the /apps endpoint under the base URL.

## Tags settings for routing rules

The Amalgam8 routing rules are tags-based. The Service Registry allows the user to
set tags through the Eureka API by using the eureka metadata configuration.

Amalgam8 tags should be defined in the eureka metadata map using the `amalgam8.tags` key
as a semicolon separated list of strings (tags).

The following configuration snippest shows how to sepcify tags in the eureka property file:

```properties
eureka.metadata.amalgam8.tags=v2;amalgam8
```

# Spring Cloud Integration

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
