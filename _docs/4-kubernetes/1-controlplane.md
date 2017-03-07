---
layout: page
title: Control Plane
permalink: /docs/kubernetes-integration-control-plane.html
redirect_from: /docs/kubernetes-integration/controlplane
category: Kubernetes Integration
order: 2
---

> Kubernetes version *__1.5.2__* or higher is recommended, as support for TPRs have improved vastly in that release.
> Earlier versions had several issues relating to TPR handling, resulting in zombie TPRs being left behind.
> If using an earlier version it may be necessary to remove TPR's before starting the controller:
>
> ```bash
> $ kubectl delete -n <namespace> thirdpartyresource routing-rule.amalgam8.io
> ```

### Deploying Amalgam8 Control Plane <a id="deploy"></a>

Amalgam8 runs a control-loop to validate routing rules stored in Third Party Resources (TPR)
 before they are fetched by the sidecars.
 The Kubernetes controller is deployed using the ReplicationController specification found in
 `examples/k8s-controlplane.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
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
spec:
  replicas: 1
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
        - name: A8_DATABASE_TYPE
          value: kubernetes
        - name: A8_DATABASE_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        ports:
        - containerPort: 8080
          name: http
---
```

To deploy:

```bash
$ kubectl create -f examples/k8s-controlplane.yaml
```

As part of its start-up sequence, the controller also registers the required extension resources into Kubernetes,
 if it isn't already registered.
 You may verify this by inspecting the pods and TPR's defined in the cluster:

```bash
$ kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
controller-r48lr       1/1       Running   0          1m
$ kubectl get thirdpartyresource
NAME                       DESCRIPTION                                    VERSION(S)
routing-rule.amalgam8.io   A specification of an Amalgam8 rule resource   v1
```


### Cleanup <a id="cleanup"></a>

To stop the control loop, run

```bash
$ kubectl delete -f examples/k8s-controlplane.yaml
```

Deleting the control loop will **not** automatically remove any rules or deregister the TPR.
These must be deleted manually.
