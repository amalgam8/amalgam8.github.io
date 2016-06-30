# Amalgam8 Microservices Fabric

## Overview

Amalgam8 is a multi-tenant microservice integration platform that works independently of any particular container platform or programming language runtime. The microservices in an Amalgam8 application can be written in any language, and run in Docker containers, virtual machines, or on a bare metal environment.

Amalgam8 provides proxy and registration sidecars, and other adaptors that are used for registering and calling
microservices. On many microservice deployment runtimes, integration with Amalgam8 requires little or no change to the microservice implementation code, because Amalgam8 is designed to make use of the benefits provided by a runtime, to provide the easiest possible integration. For example, in Kubernetes, a microservice that already has an associated
Kubernetes service definition can be registered automatically using an Amalgam8 plugin, which provides
an adapted view of the Kubernetes services in Amalgam8. Similar plugins are possible for other runtime environments. If no such plugin is available, Amalgam8 provides registration and proxy functionality by using sidecars in predefined, extensible container images.

Amalgam8 also addresses some of the challenges faced when moving to a microservices-based architecture: it provides control over the connections to integrated microservice-based applications, and the ability to test them. This includes middle-tier routing (based on percentage of traffic, user, and other criteria), and edge routing (based on using a gateway to manage incoming traffic). Delays and failures can be injected into the calls to/from microservices, enabling end-to-end resiliency testing. 

Amalgam8 is designed for extensibility and is completely available in open source.

## How it Works <a id="how-it-works"></a>

![high-level architecture](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/architecture.jpg)

At the heart of Amalgam8, are two multi-tenanted services:

1. **Registry** - A high-performance service registry that provides a centralized view of all the microservices in an application, regardless of where they are actually running.
2. **Controller** - Monitors the *Registry* and provides a REST API for registering microservice routing and other control rules, which
   the controller uses to generate and send control information to proxy servers that are running within the application.

Applications run as tenants of these two servers. The applications register their services in the registry, and use the controller to manage proxies, usually running in sidecars of the microservices.

There are 5 steps to how Amalgam8 works:

1. Microservice instances are registered in the Amalgam8 registry. There are several ways you can register them.
2. An administrator specifies routing rules and filters (e.g., version rules, test delays) to control the traffic flow between microservices.
3. The Amalgam8 controller monitors the registry and administrator input, and then generates control information that is sent to the Amalgam8 proxies.
4. Requests to microservices are made by using an Amalgam8 proxy, uusually a client-side sidecar of another microservice.
5. The Amalgam8 proxy forwards the request to an appropriate microservice, depending on the request path, headers, and the configuration that is specified by the controller.

These 5 steps are illustrated in the following diagram:

![how it works](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/how-amalgam8-works.png)

## Tenant Sidecar Process <a id="tenant-process"></a>

Amalgam8 provides a sidecar component architecture that can be configured and used by tenants in several ways.

![sidecar architecture](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/sidecar.jpg)

* **Service Registration** - Registers and heartbeats a microservice instance in the Amalgam8 registry.
* **Route Management** - Communicates with the Amalgam8 controller in the control plane to manage configuration and control-rules of a proxy.
* **Supervisor** - Manages the lifecycle of an Amalgam8 proxy server and optionally, an application microservice.
* **Proxy** - An [Nginx OpenResty](https://openresty.org/en/) server implementing the Amalgam8 proxy function.

Amalgam8 provides convenient Docker images that can be used as microservice application base images, or to access one or more of the components. However, the same components could be made available and used in other ways, such as from installation packages (deb, rpm, etc.) or as executables. Depending on the application and specific microservice, only some or all of the components might be required. For example, a microservice that requires registration, makes outgoing calls, and requires app supervision might use all of them
in a single container:

![sidecar containers](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/sidecar-containers.jpg)

In Kubernetes, however, the app supervision would probably not be used as shown in diagram (a). The app would instead
be likely running in its own separate container of a pod. For leaf microservices (i.e., microservices that make no outgoing calls), the proxy and route management components would not be needed (b). Service Registration would not be needed for services in a runtime that 
supports auto registration (c); for example, when using the amalgam8 registry adapter for Kubernetes (see below).

## Amalgam8 Service Registration <a id="service-registration"></a>

To integrate and use a microservice in Amalgam8, it needs to be registered in the Amalgam8 registry. 
The most basic way of registering, which can be used for microservices running on any platform / runtime, is shown in the following diagram:

![basic registration](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/basic-reg.jpg)

The Amalgam8 service registration sidecar component is used to register and continuously send heartbeats to the registry on
behalf of a microservice instance.
The registration sidecar keeps track of the health of the application (microservice A) and stops sending heartbeats if (or when) it terminates. If terminated, the registration in the A8 Registry will time out, and be removed. 
If there is a temporary interruption in network connectivity, the service may time out and also be removed which makes the service unavailable temporarily until the network connection to the controlplane is reestablished and the sidecar reregisters the service.
This is a common registration design pattern used in many microservice frameworks, and works for services running anywhere.

Although the above approach can be used to register any kind of service implementation,
a key design feature of Amalgam8 is to benefit from the features of any particular runtime platform that a microservice may be deployed in. For example, a microservice running in Kubernetes with a corresponding service definition,
will already have the endpoints (instances) being tracked by the kubernetes runtime (kubelet)
and is reflected in the Kubernetes service registry.

Rather than also explicitly registering this kind of service in the Amalgam8 registry, an adapter plugin can be configured
to watch the Kubernetes registry and automatically mirror the endpoints in Amalgam8:

![k8s adapter](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/k8s-adapter.jpg)

In this case, service B is an ordinary Kubernetes service defined in serviceB.yaml. There is nothing that needs to
be added or changed to integrate it with an Amalgam8 system. Instances, as they become available or unavailable, will be maintained in the Kubernetes registry by the Kubernetes runtime, and then automatically mirrored in the Amalgam8 registry by the adapter.

The Amalgam8 registry is very flexible: similar adapter plugins can be added to support other runtime environments
that already manage service registrations. For example, a Docker environment with [Weave Flux](http://weaveworks.github.io/flux/) allows services to be defined by using the fluxctl command, endpoints of which are then automatically tracked and reflected in its Etcd registry.

Integration with Amalgam8 could be done using exactly the same adapter pattern as in Kubernetes, only in this case adapting
the WeaveFlux service information in Etcd, something like this:

![weaveflux adapter](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/weaveflux-adapter.jpg)

The main point is that the Amalgam8 registry is designed to be open to these kinds of extensions, so that integrating a service into
Amalgam8 can be as easy as possible in each particular case.

## Amalgam8 Service Proxy <a id="service-proxy"></a>

All calls to Amalgam8 microservices are via an Amalgam8 proxy, allowing the controller to manage traffic to and between all the services
in the system. Most Amalgam8 proxies are sidecars of other microsystems, but there are situations where they are not.
For example, a centralized group of proxies may be appropriate in some applications or for a gateway into the
application as a whole.

The Amalgam8 proxy implementation uses [Nginx](https://nginx.org/en/)
with the [OpenResty/Lua](https://openresty.org/en/) extension as the Proxy engine as shown in the following diagram:

![a8 proxy](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/proxy.jpg)

The Route Management component of the tenant sidecar process works by communicating with the Amalgam8 controller, in the control plane, to
manage the configuration information controlling the Nginx servers. Incoming requests will then be passed to 
the appropriate Lua functions, which will route the request to an appropriate service, or version of a service,
and then load balance to an appropriate instance. 

Although Amalgam8 includes convenient Amalgam8 proxy container images with built-in
implementations of some of the most common proxy control features, the design is open and intended to allow applications
to extend the behavior by introducing their own custom routing rules and control logic.
Specific rules provided to the Amalgam8 controller can be considered opaque, and are essentially passed through to the
Amalgam8 proxies that are intended to act on them. So, by simply extending the sidecar implementation to include matching Lua
implementation code, custom rules can be specified and the system will behave as expected.

## Resilience Testing <a id="resilience-testing"></a>

A powerful feature of Amalgam8 is the ability to inject delays or failures into the calls to or between the microservices provided by Amalgam8 proxies. Isolated end-to-end testing can be run on live systems without involving the microservices themselves. You can then analyze the behavior in the log messages from the Amalgam8 proxies. 

![resilience testing](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/testing.jpg)

Using the integrated [Gremlin SDK](https://github.com/ResilienceTesting/gremlinsdk-python),
Amalgam8 users can also perform systematic resilience testing with reproducible failure scenarios and assertions to
test and debug complex microservice-based applications end-to-end.
