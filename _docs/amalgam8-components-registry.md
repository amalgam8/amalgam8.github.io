---
layout: page
title: Service Registry
permalink: /docs/registry/
order: 3
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

## Registering External Services with File-based Catalog <a id="filecatalog"></a>

Amalgam8 is typically used to control communication between internal application microservices. However, routing rules may also be injected for external cloud- and Web-based services, 
such social network sites, hosted database services that are part of the cloud platform, etc.
This extends Amalgam8's benefits, such as resiliency testing, to include the full set of services used by the application. 
To enable specifying routing rules on external services, they must first be visible via the Amalgam8 registry, by use of a static (file based) catalog type.

### File-based Catalog

The File-based catalog extension allows the registry to pull static service information from a file, in addition to supporting standard registration via REST API.
This catalog is useful when you have services that do not run the Amalgam8 protocol (e.g., a database service), and allows:

- discovery of these services using the standard Amalgam8 discovery REST API.
- specifying routing rules (such as fault injection for resiliency testing) on external services

To enable this catalog type you have to provide the `--fs_catalog` command line flag and set it to the directory of the catalog configuration files (one file per namespace).
The name of the configuration file should be `<namespace>.conf`, and it contains a single JSON object holding an array of instances.
Each instance is formatted according to the registration body defined in the [API Swagger Documentation](https://amalgam8.io/api/registry).
Service instances registered using this method are persistent and do not need to periodically heartbeat the registry in order to stay valid.

_Note: If you run multiple registry containers (e.g., for high-availability), you (probably) want to put the config files in a shared volume and to configure that volume for all the registry containers._

#### Configuration File Example

The below shows a sample configuration file adding two services: the first (`db`) with a single endpoint, and the second (`ext-service`) having two endpoints. 

```json
{
    "instances": [
        {
            "service_name": "db",
            "endpoint": {
                "type" : "http",
                "value": "www.xdb-service.url"
             },
             "tags"  : [ "v1", "us-central" ]
        },
        {
            "service_name": "ext-service",
            "endpoint": {
                "type" : "tcp",
                "value": "192.168.1.2:8080"
             }
        },
        {
            "service_name": "ext-service",
            "endpoint": {
                "type" : "tcp",
                "value": "192.168.1.2:8443"
             },
             "tags"  : [ "secure" ]
        }        
    ]
}
```

### Example Usage

The following code snippets show how to register and access an external database using Amalgam8.

#### Registring an External Service

The instance endpoint information is added in file called `default.conf` (the file name determines the registry's namespace used, in this case the external database instance is registered in the default namespace).

The sample file shows the service credentials added as metadata to the instance. This is done for illustrative purpose only, and credentials may be provided to the calling application directly. 

```json
{
   "instances": [
       {
          "service_name": "cloudant",
          "endpoint" : {
             "type" : "https",
             "value" : "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f.cloudant.com:443"
          },
          "tags" : [ "dal05" ],
          "metadata" : {
             "credentials": {
                "username": "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f",
                "password": "5b0d3f2db7401917f9ba3714ba07245f213abba3ecb96632796487f0e553435c"
             }
          }
       }
    ]
}
```

The file is stored in a location accessible to the Amalgam8 registry, to allow the registry to integrate the data on startup.
For example, the following command will load catalogs from `/amalgam8/fs_catalog/` (note that we specify the path to the _directory_ containing the file, not the full file path).

`docker run amalgam8/a8-registry:latest --fs_catalog=/amalgam8/fs_catalog/`
 
Once the registry is running, we can confirm the external service is registered successfully, and a call to 

```bash
curl amalgam8-registry:8080/api/v1/instances
```

results in:

```json
{
  "instances": [
    {
      "id": "5aff5a8eefc853d1",
      "service_name": "cloudant",
      "endpoint": {
        "type": "https",
        "value": "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f.cloudant.com:443"
      },
      "status": "UP",
      "metadata": {
        "credentials": {
           "username": "71a0c41e-6ea4-4abc-813a-5a08ad8bke3f",
           "password": "5b0d3f2db7401917f9ba3714ba07245f213abba3ecb96632796487f0e553435c"
        }
      },
      "last_heartbeat": "0001-01-01T00:00:00Z",
      "tags": [
        "dal05",
        "filesystem"
      ]
    }
  ]
}
```

#### Accessing the External Service

Once the external service instance is registered in the Amalgam8 registry, you may specify routing rules to apply on the communication path.
However, in order for the rules to be applied, communication must flow through the sidecar's proxy. The client application must call the local service URL instead of the original URL.

The recomended way (e.g., [Twelve-Factor Apps](https://12factor.net/)) is to externalize all service URL's as configuration, allowing different service endpoints to be used in development, testing and production. 
Thus, the original URL `https://71a0c41e-6ea4-4abc-813a-5a08ad8bke3f.cloudant.com:443` should be replaced in the application configuration with `http://localhost:6379/cloudant`, matching the service name specified for the instance.
The application should still provide all required headers (e.g., authorization) when contacting the sidecar. These will be passed to the upstream service unmodified.

```bash
curl -H "Authorization: basic NWIwZDNmMmRiNzQwMTkxN2Y5YmEzNzE0YmEwNzI0NWYyMTNlYmVmM2VjYjk2NjMyNzk2NDg3ZjBlNTUzNDM1Ywo=" http://localhost:6379/cloudant/<database>/<document_id>/
```
