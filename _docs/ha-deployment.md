---
layout: page
title: HA Deployment
permalink: /docs/ha/
order: 2
---

The Amalgam8 control plane can easily be deployed to support high availablility. When deployed with an external database backend, registry and controller instances are stateless, meaning any instance can fulfill requests from any client. This enables highly available and horizontally scalable deployments. 

*[Control plane architecture showing multiple registry and controller instances with database backend]*

# Principles

Deploying a highly available Amalgam8 control plane requires the following:

* Multiple control plane container instances to provide redundancy.
* Load balancer(s) to distribute requests to the control plane containers.
* An external persistent storage backend such as Redis. The control plane supports in-memory storage for development, but this cannot be used for deployments with multiple controller instances because each instance maintains its own copy of the data. In addition, no data is persisted with the in-memory store, which is not practical for real deployments.

# Kubernetes

The simplest way to run a local HA deployment of the Amalgam8 control plane is with Kubernetes. Kubernetes supports collections of containers grouped into a services. Requests to Kubernetes services are load balanced to their constituent containers, which satisfies the requirements for load balancing to multiple redundant control plane containers. 

Assuming Kubernetes is installed, create the following Kubernetes configuration file and save it as `controlplane.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    name: redis
spec:
  ports:
  - port: 6379
    targetPort: 6379
    nodePort: 31400
    protocol: TCP
  selector:
    name: redis
  type: NodePort
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis
  labels:
    name: redis
spec:
  replicas: 1
  selector:
    name: redis
  template:
    metadata:
      labels:
        name: redis
    spec:
      containers:
      - name: redis
        image: redis:alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: controller
  labels:
    name: controller
spec:
  ports:
  - port: 6080
    targetPort: 8080
    nodePort: 31200
    protocol: TCP
  selector:
    name: controller
  type: NodePort
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: controller
  labels:
    name: controller
spec:
  replicas: 3
  selector:
    name: controller
  template:
    metadata:
      labels:
        name: controller
    spec:
      containers:
      - name: controller
        image: amalgam8/a8-controller
        imagePullPolicy: IfNotPresent
        env:
        - name: A8_LOG_LEVEL
          value: info
        - name: A8_DATABASE_TYPE
          value: redis
        - name: A8_DATABASE_HOST
          value: redis://$(REDIS_SERVICE_HOST):$(REDIS_SERVICE_PORT)
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: registry
  labels:
    name: registry
spec:
  ports:
  - port: 5080
    targetPort: 8080
    nodePort: 31300
    protocol: TCP
  selector:
    name: registry
  type: NodePort
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: registry
  labels:
    name: registry
spec:
  replicas: 3
  selector:
    name: registry
  template:
    metadata:
      labels:
        name: registry
    spec:
      containers:
      - name: registry
        image: amalgam8/a8-registry
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        env:
        - name: A8_STORE
          value: redis
        - name: A8_STORE_ADDRESS
          value: $(REDIS_SERVICE_HOST):$(REDIS_SERVICE_PORT)
```

Deploy the control plane:

```bash
kubectl create -f controlplane.yaml
```

This command will start an Amalgam8 control plane with a Redis storage backend and 3 instances each of the controller and the registry. Requests to the controller and registry will be load balanced to the instances. If one of the control plane containers becomes unhealthy, Kubernetes halt routing to the container and will restart the container.

HA deployments of the Amalgam8 control plane in other environments will differ in specific steps, but follow the same principles outline above.
