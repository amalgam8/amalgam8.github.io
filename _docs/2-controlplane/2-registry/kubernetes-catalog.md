---
layout: page
title: Kubernetes Catalog
permalink: /docs/control-plane/registry/kubernetes-catalog/
category: Control Plane
subcategory: Service Registry
order: 4
---

A microservice running in Kubernetes with a corresponding service
definition, will already have the endpoints (instances) being tracked by
the Kubernetes runtime (kubelet) and reflected in the Kubernetes service
registry. Instead of using the sidecar to explicitly register this
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

