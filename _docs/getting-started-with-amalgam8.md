---
layout: page
title: Getting Started
permalink: /docs/getting-started/
order: 1
---

Running your application on top of the Amalgam8 microservice platform
involves two main steps: (a) setting up the control plane services
(Amalgam8 service registry and Amalgam8 route controller), and (b) using the
Amalgam8 sidecar for communication between microservices instead of direct
microservice-to-microservice communication. 

The control plane setup, i.e., step (a) is needed only if you are running
Amalgam8 locally. If the Amalgam8 control plane is provided as a service, you
need to obtain the URLs and authentication tokens for the controller and
the registry.

The Amalgam8 sidecar enables intelligent request routing while automating
service registration, discovery, and client-side load-balancing. The
Amalgam8 sidecar is based on Go+Nginx and follows an architecture similar
to
[Netflix Prana sidecar](http://techblog.netflix.com/2014/11/prana-sidecar-for-your-netflix-paas.html)
or
[AirBnB Smartstack](http://nerds.airbnb.com/smartstack-service-discovery-cloud/).


## TL;DR

* Start the control plane services (Amalgam8 Controller and Registry)

```bash
    #registry
    docker run -d amalgam8/a8-registry
    #controller
    docker run -d amalgam8/a8-controller
```


* Install the Amalgam8 CLI (`a8ctl`)

```bash
    sudo pip install git+https://github.com/amalgam8/a8ctl
```


* Install the sidecar in your Dockerized microservice (assuming you have
  `curl` pre-installed)

```Dockerfile
    RUN curl -sSL https://github.com/amalgam8/amalgam8/releases/download/${VERSION}/a8sidecar.sh | sh
```


* Launch your app via the sidecar

```Dockerfile
    ENTRYPOINT ["a8sidecar", "--proxy", "--register", "--supervise", "YOURAPP", "YOURAPP_ARG", "YOURAPP_ARG"]
```

* Inject environment variables into your container


```bash
    A8_SERVICE=service_name:service_tags
    A8_ENDPOINT_PORT=port_where_service_is_listening
    A8_ENDPOINT_TYPE=http|https
    A8_REGISTRY_URL=http://a8registryURL
    A8_CONTROLLER_URL=http://a8controllerURL
```

* Make API calls to other microservices via the sidecar [http://localhost:6379/serviceName/endpoint]()

* Control traffic to microservices using the control plane API or the
   [a8ctl](https://github.com/amalgam8/a8ctl) utility

```bash
    a8ctl route-set serviceName --default v1 --selector 'v2(user="Alice")' --selector 'v3(user="Bob")'
```

For detailed instructions on how to integrate your application with the
sidecar, please refer to the
[sidecar documentation](/docs/amalgam8-components-sidecar/)

## Walkthoughs

The Amalgam8 repository on GitHub has
[detailed walkthroughs](https://github.com/amalgam8/amalgam8/tree/master/examples)
for setting up the Amalgam8 control plane as well as demo applications that
illustrate different features of Amalgam8.
