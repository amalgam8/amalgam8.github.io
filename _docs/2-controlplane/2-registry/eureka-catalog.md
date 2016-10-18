---
layout: page
title: Eureka Catalog
permalink: /docs/control-plane/registry/eureka-catalog/
category: Control Plane
subcategory: Service Registry
order: 3
---

A microservice running on the [Netflix stack](https://netflix.github.io/) will already
have the instances being tracked by the [Netflix Eureka server](https://github.com/Netflix/eureka).
Instead of using a sidecar to explicitly register this
service instance once again in the Amalgam8 registry, the Eureka registry adapter
can be configured to watch the Eureka server and
automatically mirror the instances in Amalgam8 as shown in the figure below.

![eureka adapter](/docs/figures/amalgam8-registry-eureka-adapter.png)

In the above figure, service B is running on the Netflix stack and is registered 
in the Eureka server. Instances running on the Netflix stack, as they come and go, will
be maintained in the Eureka server by the Netflix stack, and then automatically
mirrored in the Amalgam8 registry by the eureka-adapter.

## Using the Eureka Catalog Extension

The Eureka catalog extension provides a registry adapter that allows the Amalgam8 registry to mirror
service instance information in the Eureka registry, in addition to supporting standard registration
via the REST API. 

To enable the Eureka catalog type you have to provide the `--eureka_url` command
line flag (or `A8_EUREKA_URL` environment variable) when launching the registry.

The eureka url is the endpoint of the API server for the Netflix Eureka server being mirrored (multiple
servers in the same cluster can be specified using multiple `--eureka_url` command line flags).

```
A8_EUREKA_URL=http://eureka-host:8080/eureka/v2
```
