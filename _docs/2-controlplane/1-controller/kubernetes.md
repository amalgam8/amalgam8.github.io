---
layout: page
title: Kubernetes
permalink: /docs/control-plane-controller-kubernetes.html
redirect_from: /docs/control-plane/controller/kubernetes
category: Control Plane
subcategory: Route Controller
order: 2
---

> Kubernetes version *__1.5.2__* or higher is recommended, as support for TPRs have improved vastly in that release.
> Earlier versions had several issues relating to TPR handling, resulting in zombie TPRs being left behind.
> If using an earlier version it may be necessary to remove TPR's before starting the controller:
>
> ```bash
> $ kubectl delete -n <namespace> thirdpartyresource routing-rule.amalgam8.io
> ```

### Prerequisites and Caveats <a id="prerequisites-caveats"></a>

*__Before__* beginning working through the examples and demos, please confirm all container images for the control plane,
 [helloworld](docs/demo-helloworld.html) and [bookinfo](/docs/demo-bookinfo.html) demo applications, are stored in a
 docker image registry accessible to the Kubernetes cluster.
 By default, the latest images are hosted in the `amalgam8` docker hub image repository, but if you're
 building a different version, be sure to push the modified images and change the resource manifests accordingly.

 Once the below `make` completes, tag and push the newly built images to an image repository accessible
 to the Kubernetes cluster. It is further advised that `imagePullPolicy` is set to `Always` in all yaml
 files, to ensure the latest are indeed used in testing.

 ```bash
 $ git clone git@github.com:amalgam8/amalgam8.git
 $ cd amalgam8
 # trick makefile to generate all with a "latest" version tag by setting version to "vlatest"
 $ APP_VER_ABBR=vlatest make build.controller build.sidecar build.exampleapps dockerize.sidecar.envoy.ubuntu dockerize.controller
 ```


### Native Kubernetes Integration

When deployed on [Kubernetes](https://kubernetes.io), Amalgam8 provides a near native experience
 by reusing native Kubernetes tools and fucntions to run and manage its control plane components.
 The [Service registration and discovery](/docs/control-plane-registry.html) functionality leverages Kubernetes
 built in kubernetes service registry and discovery, and the [Route Controller](/docs/control-plane-controller.html) uses
 Kubernetes [Third Party Resources](https://kubernetes.io/docs/user-guide/thirdpartyresources/)
 to store routing rules, thus enabling familiar access using `kubectl` command line.

### Sidecar Configuration <a id="sidecar-config"></a>

The Amalgam8 sidecar configuration need to change to match the fact that it is running in
 in a Kubernetes cluster. Please refer also to [sidecar configuration](/docs/sidecar-configuration.html).

 - Service instance registration is not required, pods are automatically registered by Kubernetes.
 - The `discovery_adapter` and `rules_adapter` flags should be set to `kubernetes`
 - The access URLs and token are not required (they are automatically retrieved from the pod).
  The default, in-cluster, configuration may be overridden by setting the `kubernetes_url` and
  `kubernetes_token` flags.
 - The sidecar will fetch rules and service instance information from the `default` namespace.
   This may be overridden by setting the `kubernetes_namespace` flag.
 - Service name and labels may be specified using the `service` flag, as before. Alternately,
   if the `kubernetes_pod_name` is set, the source service name and tags are automatically
   retrieved from the pod's service association and labels. Note that if a pod is mapped to
   multiple Kubernetes service, an arbitrary service identity will be selected from the list.
   Tags are automatically generated from the pod's labels by converting each label to a tag
   whose format is `key=value` (or simple `key` if value is undefined). The `key=value` format
   should be used in defininig routing rules.
 - Kubernetes service names are used directly in defining rule target services.
