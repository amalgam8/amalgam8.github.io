---
layout: page
title: Introduction
permalink: /docs/kubernetes-integration-intro.html
redirect_from: /docs/kubernetes-integration/
category: Kubernetes Integration
order: 0
---


When deployed on [Kubernetes](https://kubernetes.io), Amalgam8 provides a near native experience
 by reusing native Kubernetes tools and fucntions to run and manage its control plane components.
 The [Service Registry](/docs/control-plane-registry.html) functionality leverages Kubernetes
 services, and the [Route Controller](/docs/control-plane-controller.html) uses
 Kubernetes [Third Party Resources](https://kubernetes.io/docs/user-guide/thirdpartyresources/)
 to store routing rules, thus enabling familiar access using `kubectl` command line.

*__Before__* begining working through the examples and demos, pelase confirm all container images for the control plane,
 [helloworld](docs/demo-helloworld.html) and [bookinfo](/docs/demo-bookinfo.html) demo applications, are stored in a
 registry accessible to the cluster.
 By default, the latest images are hosted in the `amalgam8` docker hub image repository, but if you're
 building a different version, be sure to push the modified images and change the resource manifests accordingly.

### Sidecar Configuration <a id="sidecar-config"></a>

The sidecar and proxy run as a separate container in the same Kubernetes pod as the application code.
 They are no longer required to run in the same container, which greatly simplifies integration (e.g.,
 the bundling only happens at deployment time, via the pod manifest, and not during image creation)

The Amalgam8 sidecar configuration need to change to match the fact that it is running in
 in a Kubernetes cluster. Please refer also to [sidecar configuration](/docs/sidecar-configuration.html).

 - The `discovery_adapter` and `rules_adapter` flags should be set to `kubernetes`
 - Service instance registration is not required, pods are automatically registered by Kubernetes.
 - The access URLs and token are not required (they are automatically retrieved from the pod).
  The default, in-cluster, configuration may be overridden by setting the `kubernetes_url` and
  `kubernetes_token` flags.
 - The sidecar will fetch rules and service instance information from the `default` namespace.
   This may be overridden by setting the `kubernetes_namespace` flag.
 - If the `kubernetes_pod_name` is set, the source service name and tags are automatically
   retrieved from the pod's service association and labels. Note that if a pod is mapped to
   multiple Kubernetes service, an arbitrary service identity will be selected from the list.
   Tags are automatically generated from the pod's labels by converting each label to a tag
   whose format is `key=value` (or simple `key` if value is undefined). The `key=value` format
   should be used in defininig routing rules.
 - Kubernetes service names are used directly in defining rule target services.
