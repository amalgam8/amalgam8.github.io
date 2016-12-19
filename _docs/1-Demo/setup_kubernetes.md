---
layout: page
title: Setup
permalink: /docs/demo-setup.html
redirect_from: /docs/demo/setup/
category: Demo
order: 1
---

## Before you begin

If you don't already have a Kubernetes environment, you can install kubernetes locally by using [Minikube](https://github.com/kubernetes/minikube).

## Setting up the Control Plane

The Amalgam8 Control Plane consists of two components: the
[service registry](/docs/control-plane-registry.html) and the
[route controller](/docs/control-plane-controller.html).  In addition to these two
components, for the purposes of this demo, a stock ELK stack is also
included as part of the control plane deployment scripts, to collect logs
from the microservices in the application.

The Amalgam8 Control Plane contains an ELK stack.
Elasticsearch 5.0 [requires increasing the max map count](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html)
for certain linux environments such as Vagrant VMs, Docker Toolbox and Minikube.
If you are running any of these tools, run the following command on those VMs:

```
bash
sudo sysctl -w vm.max_map_count=262144
```

SSH into the minikube virtual machine by running `minikube ssh`, and run:

```
bash
kubectl create -f examples/k8s-controlplane.yaml
```

## Setting up the Amalgam8 CLI

The `a8ctl` command line utility provides a convenient way to setup and
manage routes across microservices as well as inspect the state of the
system. The CLI can be installed via the `pip` tool (available as part of
standard Python installation) directly from Python's package repository or
from `a8ctl` [github repository](https://github.com/amalgam8/a8ctl).

1. Run the folowing command to install the a8ctl into your home directory:

```
bash
pip install --user a8ctl
```

1. Add the location of the `a8ctl` command to the `PATH` environment
variable. On OS X, run

```
bash
export PATH=$PATH:${HOME}/Library/Python/2.7/bin
```

On Linux, run

```
bash
export PATH=$PATH:${HOME}/.local/bin
```

To use the Amalgam8 CLI (`a8ctl`) for the demo walkthroughs we need
to setup two environment variables: `A8_CONTROLLER_URL` and
`A8_REGISTRY_URL`. These variables should point to the publicly accessible
REST endpoints exposed by the Amalgam8 controller and registry respectively. The value of these variables depends on the environment where you setup the
control plane.

1. On minikube, run:

```
bash
export A8_CONTROLLER_URL=http://$(minikube ip):31200
export A8_REGISTRY_URL=http://$(minikube ip):31300
```

Now that you've configured Amalgam8 for Kubernetes, let's move onto running the demo.
