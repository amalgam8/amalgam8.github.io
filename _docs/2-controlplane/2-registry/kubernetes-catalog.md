---
layout: page
title: Kubernetes Catalog
permalink: /docs/control-plane/registry/kubernetes-catalog/
category: Control Plane
subcategory: Service Registry
order: 5
---

A microservice running in Kubernetes with a corresponding service
definition, will already have the endpoints (instances) being tracked by
the Kubernetes runtime (kubelet) and reflected in the Kubernetes service
registry. Instead of using a sidecar to explicitly register this
service once again in the Amalgam8 registry, the Kubernetes registry adapter
can be configured to watch the Kubernetes registry and
automatically mirror the endpoints in Amalgam8 as shown in the figure below.

![k8s adapter](/docs/figures/amalgam8-registry-k8s-adapter.svg)

In the above figure, service B is a Kubernetes service defined in
serviceB.yaml. Instances, as they come and go, will
be maintained in the Kubernetes registry by the Kubernetes runtime, and
then automatically mirrored in the Amalgam8 registry by the
kubernetes-adapter.

## Using the Kubernetes Catalog Extension

The Kubernetes catalog extension provides a registry adapter that allows the Amalgam8 registry to mirror
service information in the Kubernetes registry, in addition to supporting standard registration
via the REST API. 

To enable the Kubernetes catalog type you have to provide the `--k8s_url` command
line flag (or `A8_K8S_URL` environment variable) and, if running in a secure environment,
a corresponding `--k8s_token` (or `A8_K8S_TOKEN`) value, when launching the registry.

The k8s url is the endpoint of the API server for the Kubernetes runtime being mirrored.
When deploying the registry (control plane) in Kubernetes itself, the url can simply be set using
the environment variables provided by the Kubernetes runtime.

```
A8_K8S_URL=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
```

## Service Definitions

Services registered using the Kubernetes registry adapter are defined in Kubernetes in the
usual way using a [Service](http://kubernetes.io/docs/user-guide/services/) definition,
although a few conventions must be followed to enable version and content-aware routing in Amalgam8.

As you might expect, the Kubernetes registry adapter simply uses the 
Kubertnetes service name for the corresponding service name in the Amalgam8 registry.
For example, the following Kubernetes service will produce a service named "helloworld"
in the Amalgam8 registry.

```
apiVersion: v1
kind: Service
metadata:
  name: helloworld
spec:
  ...
```

Version tags associated with the running instances of the service are extracted from the
[Replication Controllers](http://kubernetes.io/docs/user-guide/replication-controller/)
managing the associated pods. The current adapter implementation uses the name of
the replication controller to extact a single version tag for the associated registrations.
If the replication controller name includes dash ("-") characters, the version tag is
the substring after the last one. Otherwise the full name is used.

The following example will produce two instances of "helloworld:v1" and
two instances of "helloworld:v2" in the Amalgam8 registry.

```
apiVersion: v1
kind: Service
metadata:
  name: helloworld
spec:
  selector:
    service: helloworld
  ports:
    ...
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: helloworld-v1
spec:
  replicas: 2
  selector:
    name: helloworld-v1
  template:
    metadata:
      labels:
        name: helloworld-v1
        service: helloworld
    spec:
      containers:
        ...
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: helloworld-v2
spec:
  replicas: 2
  selector:
    name: helloworld-v2
  template:
    metadata:
      labels:
        name: helloworld-v2
        service: helloworld
    spec:
      containers:
        ...
```

You can see the correpsonding Amalgam8 services by running the `a8ctl service-list` command.

```
$ a8ctl service-list
+------------+------------------------------------+
| Service    | Instances                          |
+------------+------------------------------------+
| helloworld | v1,kubernetes(2), v2,kubernetes(2) |
+------------+------------------------------------+
```

Notice that in addition to the expected "v1" and "v2" version tags, the instances also have
a second tag, "kubernetes". Every registration created by the Kubernetes adapter will include
this extra tag, indicating that the registration is derived from a Kubernetes service.

### Endpoint Type

The type of endpoint in an Amalgam8 service registration is derived from the protocol
and name fields of the corresponding Kubernetes service port. 
In Kubernetes, the protocol can be either "TCP" or "UDP", which map to Amalgam8 
endpoint types "tcp" and "udp", respectively.

Consider the following service definition.

```
apiVersion: v1
kind: Service
metadata:
  name: helloworld
spec:
  selector:
    service: helloworld
  ports:
  - protocol: TCP
    ...
```

The instances of this service in the Amalgam8 registry will look something like this:

```
   {
     "tags": [
       "v1",
       "kubernetes"
     ],
     "endpoint": {
       "value": "172.17.0.11:5000",
       "type": "tcp"
     },
     "service_name": "helloworld",
     ...
   }
```

This works fine in Amalgam8, however, to enable Amalgam8's full content-aware routing features
(e.g., header matching rules), the endpoint type in Amalgam8 needs to be "http" or "https",
instead of simple "tcp".
This can be done by setting the name of the corresponding port to the desired type.

```
apiVersion: v1
kind: Service
metadata:
  name: helloworld
spec:
  selector:
    service: helloworld
  ports:
  - protocol: TCP
    name: http
    ...
```

This will produce the same instance registration in Amalgam8 only with endpoint type "http", instead of "tcp".

```
   {
     "tags": [
       "v1",
       "kubernetes"
     ],
     "endpoint": {
       "value": "172.17.0.11:5000",
       "type": "http"
     },
     "service_name": "helloworld",
     ...
   }
```
