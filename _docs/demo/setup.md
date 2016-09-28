---
layout: page
title: Setup
permalink: /docs/demo/setup/
category: Demo
order: 1
---

While Amalgam8 supports multi-tenancy, for the sake of simplicity, this
demo will setup Amalgam8 in single-tenant mode.

## Download

Download and unzip the latest release of Amalgam8 demos from the
[Amalgam8 Releases](https://www.github.com/amalgam8/amalgam8/releases)
page. You should find a folder named `examples` after extracting files from
the zip archive.

## Requirements

* [Docker](https://www.docker.com/products/docker#/)
  You need `docker` 1.10 or higher and `docker-compose` 1.5.1 or higher.

* [Python 2.7.9](https://www.python.org/downloads/) or higher. If you are
  using Linux or Mac OS X, you should already have the requisite version of
  Python in your system.

### Additional Requirements

* If you are planning to use Kubernetes, then you need a
  working kubernetes installation. To install kubernetes locally, checkout
  [minikube](https://github.com/kubernetes/minikube).
  
  * If you are using Google Cloud Platform, setup the
    [Google Cloud SDK](https://cloud.google.com/sdk/).

* If you are planning to run the demos on IBM Bluemix, you need to install
  the [Bluemix CLI](http://clis.ng.bluemix.net/ui/home.html) version 0.4.1
  or later with the IBM-Containers plugin 0.5.800 or later.

With the requirements out of the way, lets move on to the next stage:
running Amalgam8.

## Setting up the Control Plane

The Amalgam8 Control Plane consists of two components: the
[service registry](/docs/registry/) and the
[route controller](/docs/controller/).  In addition to these two
components, for the purposes of this demo, a stock ELK stack is also
included as part of the control plane deployment scripts, to collect logs
from the microservices in the application.

The commands to deploy the control plane for different environments are as
follows:

_Docker Compose_
  
```bash
docker-compose -f examples/docker-controlplane.yaml up -d
```

_Kubernetes_ on localhost or on Google Cloud

```bash
kubectl create -f examples/k8s-controlplane.yaml
```

_IBM Bluemix_

1. Login to Bluemix and initialize the container environment using 

   ```
   bluemix login
   bluemix ic init
   ```

1. Create Bluemix routes (DNS names) for the registry, controller and the bookinfo app's gateway:  

   ```bash
   cf create-route <your bluemix space> mybluemix.net -n <your registry route>
   cf create-route <your bluemix space> mybluemix.net -n <your controller route>
   ```

   where `<your bluemix space>` is the name of your Bluemix space and
   `<your route ...>` is a unique route name for `registry` and
   `controller`. For example `my-space-name-a8-registry` and 
   `my-space-name-a8-controller`. Make a note of the route names you
   choose for the next step.


1. Customize the _examples/bluemix.cfg_ in the downloaded folder as follows:
    * BLUEMIX_REGISTRY_NAMESPACE should be your Bluemix registry namespace
      obtained from the folowing command:

      ```bash
      bluemix ic namespace-get
      ```

    * REGISTRY_HOSTNAME should be the route name assigned to the registry in the previous step

    * CONTROLLER_HOSTNAME should be the route name assigned to the controller in the previous step

1. Deploy the control plane services (registry and controller) on bluemix.

   ```bash
   examples/a8-bluemix create controlplane
   ```

   Verify that the controller and registry are running using the following commands: 

   ```bash
   bluemix ic groups
   ```
 
   You should see the groups `amalgam8_controller` and `amalgam8_registry` listed in the output.

## Setting up the Amalgam8 CLI

The `a8ctl` command line utility provides a convenient way to setup and
manage routes across microservices as well as inspect the state of the
system. The CLI can be installed via the `pip` tool (available as part of
standard Python installation) directly from Python's package repository or
from `a8ctl` [github repository](https://github.com/amalgam8/a8ctl).

Run the folowing command to install the a8ctl into your home directory:

```bash
pip install --user a8ctl
```

Add the location of the `a8ctl` command to the `PATH` environment
variable. On OS X, run

```bash
export PATH=$PATH:${HOME}/Library/Python/2.7/bin
```

On Linux, run

```bash
export PATH=$PATH:${HOME}/.local/bin/a8ctl
```

To use the Amalgam8 CLI (`a8ctl`) for the demo walkthroughs we need
to setup two environment variables: `A8_CONTROLLER_URL` and
`A8_REGISTRY_URL`. These variables should point to the publicly accessible 
REST endpoints exposed by the Amalgam8 controller and registry respectively.

The value of these variables depends on the environment where you setup the
control plane.

_Docker Compose_

```bash
export A8_CONTROLLER_URL=http://localhost:31200
export A8_REGISTRY_URL=http://localhost:31300
```

_Kubernetes_

On minikube

```bash
export A8_CONTROLLER_URL=http://$(minikube ip):31200
export A8_REGISTRY_URL=http://$(minikube ip):31300
```

On Google Cloud Platform, assign a public IP to the Node running the
controller and the registry, and set the environment variables
respectively.

_IBM Bluemix_

```bash
export A8_CONTROLLER_URL=http://<controller route>.mybluemix.net
export A8_REGISTRY_URL=http://<registry route>.mybluemix.net
```

where `<controller route>` and `<registry route>` correspond to the routes
that you set for the controller and registry respectively.
