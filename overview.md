# Amalgam8 Microservices Fabric

## Overview

Amalgam8 is a multi-tenanted, platform and runtime agnostic, microservices integration framework.
The microservices in an Amalgam8 application can be polyglot, run in containers, VMs, or even on bare metal.

Amalgam8 provides proxy and registration sidecars and other adapter mechanisms that are used for registering and calling
microservices. Depending on a microservice's deployment runtime, integration with Amalgam8 can require little or no significant change
to the microservice impelementation code because Amalgam8 is designed to leverage the benefits of any given runtime
and then provide the easiest possible integration for each. For example, in Kubernetes, a microservice that adready has an associated
Kubernetes service definition can be registered automatically using an Amalgam8 plugin providing
an adapted view of the Kubernetes services in Amalgam8. 
Similar plugins are possible for other runtime environments, but where they are not, Amalgam8 also provides registration
and proxy functionality using sidecars in a number of predefined, but extensible, container images.

In addition to ease of integration, Amalgam8 also provides the integrated microservice-based applications with core functionality
enabling a great deal of control and testability, addressing some of the most challenging issues faced when moving to
a microservices-based system. This includes edge and mid-tier version-routing based on traffic percentage, user, and other
criteria. Delays and failures can also be injected into the path of calls to and between microservices, enabling advanced
end-to-end resiliency testing. And most importantly, all of this is designed for extensibility and is completely available in open source.

## How it Works

![high-level architecture](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/architecture.jpg)

At the heart of Amalgam8, are two mutli-tenanted services:

1. **Registry** - A high performance service registry that provides a centralized view of all the microservices in an application, regardless
   of where they are actually running.
2. **Controller** - Monitors the Registry and provides a REST API for registering routing and other microservice control-rules, which
   it uses to generate and send control information to proxy servers running within the application.

Application run as tenants of these two servers. They register their services in the Registry and use the Controller to manage proxies,
usually running in sidecars of the microservices.

![how it works](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/how-it-works.jpg)

1. Microservice instances are registered in the A8 Registry. There are several ways this may be accomplished (see below).
2. Administrator specifies routing rules and filters (e.g., version rules, test delays) to control traffic flow between microservices.
3. A8 Controller monitors the A8 Registry and administrator input and then generates control information that is sent to the A8 Proxies.
4. Requests to microservices are via an A8 Proxy (usually a client-side sidecar of another microservice)
5. A8 Proxy forwards request to an approriate microservice, depending on the request path and headers and the configuration specified by the controller.

## Tenant Sidecar Process

Almagam8 provides a sidecar component architecture that can be configured and used by tenants in a number of ways.

![sidecar architecture](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/sidecar.jpg)

* **Service Registration** - Registers and heartbeats a microservice instance in the Amalgam8 registry.
* **Route Management** - Communicates with the A8 Controller in the control plane to manage configuration and control-rules of an A8 Proxy.
* **Supervisor** - Manages the lifecycle of an A8 Proxy server and optionally an application microservice itself.
* **Proxy** - An [Nginx OpenResty](https://openresty.org/en/) server implementing the A8 Proxy function.

Since most microservices run in some kind of container environment, Amalgam8 provides convenient Docker images that can be used
as microservice application base images or simply to access one or more of the components. 
The same components could, howoever, be
made available and used in other ways such as from installation packages (deb, rpm, etc.) or plain executables.

Depending on the application and specific microservice, only some or all of the components may be needed. 
For example, a microservice that requires registration, makes outgoing calls, and requires app supervision might use all of them
in a single container:

![sidecar containers](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/sidecar-containers.jpg)

In Kubernetes, however, the app supervision would probabaly not be used as shown in diagram (a). The app would instead
be likely running in its own separate container of a pod. For leaf microservices (i.e., ones that make no outgoing calls), the proxy and
route management components would not be needed (b). Service Registration would not be needed for services in a runtime that 
supports auto registration (c), for example, when using the amalgam8 registry adapter for Kubernetes (see below).

## Amalgam8 Service Registration

To integrate and use a microservice in Amalgam8, it needs to be registered in the A8 Registry. 
The most basic way of registering, which can be used for microservcies running on any platform / runtime, is shown in the following diagram:

![basic registration](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/basic-reg.jpg)

The A8 service registration sidecar component is used to register and continuously send hartbeats to the registry on
behalf of a microservice instance.
The registration sidecar keeps track of the health of the application (microservice A) and stops sending heartbeats if or when it terminates,
soon after which the registration in the A8 Registry will time out and be removed. 
If there is a temporary interuption in network connectivity, the service may time out and also be removed (making it unavailable temporarily)
until the network connection to the controlplane is reestablished and the sidecar reregisters the service.
This is a common registration design pattern used in many microservice frameworks, and will work for services running anywhere.

Although the above approach can be used to register any kind of service implementation,
a key design feature of Amalgam8 is to leverage features of any particular runtime platform that a microservice may be deployed in.
For example, a microservice running in Kubernetes with a corresponding service definition,
will already have the endpoints (instances) being tracked by the kubernetes runtime (kubelet)
and reflected in the Kubernetes service registry.

Rather than also explicitly registering this kind of service in the A8 Registry, an adapter plugin can be configured
to watch the Kubernetes registry and automatically mirror the endpoints in Amalgam8:

![k8s adapter](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/k8s-adapter.jpg)

In this case, service B is an ordinary Kubernetes service defined in serviceB.yaml. There is nothing that needs to
be added or changed to integrate it with an Amalgam8 system. Instances, as they come and go, will be maintained in the Kubernetes
registry by the kubernetes runtime, and then automatcally mirrored in the A8 registry by the adapter.

The Amalgam8 registry is very flexible in this regard. Similar adapter plugins can be added to support other runtime environments
that already manage service registrations.
For example, a Docker environment with [Weave Flux](http://weaveworks.github.io/flux/) allows services to be defined
using the fluxctl command, endpoints of which are then automatically tracked and reflected in its Etcd registry.

Integration with Amalgam8 could be done using exactly the same adapter pattern as in Kubernetes, only in this case adapting
the WeaveFlux service information in Etcd, something like this:

![weaveflux adapter](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/weaveflux-adapter.jpg)

The main point is that the A8 registry is designed to be open to these kinds of extensions, so that integrating a service into
Amalgam8 can be as easy as possible in each particular case.

## Amalgam8 Service Proxy

All calls to Amalgam8 microservices are via an A8 Proxy, allowing the A8 controller to manage traffic to and beteen all the services
in the system. Most A8 Proxies are sidecars of other microsystems, but there are situations where they are not.
For example, a centralized group of proxys may be appropriate in some applications or for a gateway into the
application as a whole.

The A8 Proxy implementation uses Nginx with the OpenResty/Lua extension as the Proxy engine as shown in the following diagram:

![a8 proxy](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/proxy.jpg)

The Route Management component of the tenant sidecar process works by communicating with the A8 Controller, in the control plane, to
manage the configuration information controlling the NGinx servers. Incoming requests will then be passed to 
the appropriate Lua functions, which will route the request to an approriate service, or version of a service,
and then load balance to an approriate instance. 

A point worth highlighting is that although Amalgam8 includes convenient A8 Proxy container images with built-in
implmentations of some of the most common proxy control features, the design is open and intended to allow applications
to extend the behavior by introducing their own custom routing rules and control logic.
Specific rules provided to the A8 Controller can be considered opaque and will essentially be passed though to the
A8 Proxy's that are intended to act on them. So, by simply extending the sidecar implementation to include matching Lua
implementation code, custom rules can be specified and the system will behave as expected.

## Resilience Testing

The ability to inject and affect the calls to or between microservices provided by A8 proxies,
enables another very powerful feature of Amalgam8.
Isolated end-to-end testing can be run on live systems without involving the microservices themselves.
Amalgam8 allows you to inject random delays or failures into the call path between microservices
and then analyze the behavior provided by log messages from the A8 proxies. 

![resilience testing](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/testing.jpg)

Using the integrated [Gremlin SDK](https://github.com/ResilienceTesting/gremlinsdk-python),
Amalgam8 users can also perform systematic resilience testing with reproducible failure scenarios and assertions to
test and debug complex microservice-based applications end to end.
