---
layout: page
title: Setup
permalink: /docs/demo/setup/
category: Demo
order: 1
---

While Amalgam8 supports multi-tenancy, for the sake of simplicity, this
demo will setup Amalgam8 in single-tenant mode.

## Requirements

* [Docker](https://www.docker.com/products/docker#/)
  You need `docker` 1.10 or later and `docker-compose` 1.5.1 or later.

* Amalgam8 Python CLI. On Linux and Mac OS X, run the folowing command:

  ```bash
  sudo pip install git+https://github.com/amalgam8/amalgam8/a8ctl
  ```

### Additional Requirements

* If you are planning to use Kubernetes, then you need a
  working kubernetes installation. To install kubernetes locally, checkout
  [minikube](https://github.com/kubernetes/minikube).
  
  * If you are using Google Cloud Platform, setup the
    [Google Cloud SDK](https://cloud.google.com/sdk/).

* If you are planning to run the demos on IBM Bluemix, you need to have
  the [CF CLI 6.12.0 or later](https://github.com/cloudfoundry/cli/releases), and
  the [Bluemix CLI 0.3.3 or later](https://clis.ng.bluemix.net/).

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
docker-compose -f <url_to_a8_controlplane.yml> up -d
```

_Kubernetes_ on localhost or on Google Cloud

```bash
kubectl create -f <url_to_a8_controlplane.yaml>
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


1. Download the scripts to automate Bluemix deployment from this
   [here](https://url_to_bluemix_scripts).
   
1. Customize the _.bluemixrc_ in the downloaded folder as follows:
    * BLUEMIX_REGISTRY_NAMESPACE should be your Bluemix registry namespace
      obtained from the folowing command:

      ```bash
      bluemix ic namespace-get
      ```

    * REGISTRY_HOSTNAME should be the route name assigned to the registry in the previous step

    * CONTROLLER_HOSTNAME should be the route name assigned to the controller in the previous step

1. Deploy the control plane services (registry and controller) on bluemix.

   ```bash
   ./a8-bluemix create controlplane
   ```

   Verify that the controller and registry are running using the following commands: 

   ```bash
   bluemix ic groups
   ```
 
   You should see the groups `amalgam8_controller` and `amalgam8_registry` listed in the output.

## Configuring the a8ctl CLI

To use the Amalgam8 CLI (`a8ctl`) for the demo walkthroughs we need
to setup two environment variables: `A8_CONTROLLER_URL` and
`A8_REGISTRY_URL`. These variables should point to the publicly accessible 
REST endpoints exposed by the Amalgam8 controller and registry respectively.

The value of these variables depends on the environment where you setup the
control plane.

_Docker Compose_

```bash
A8_CONTROLLER_URL=http://localhost:31200
A8_REGISTRY_URL=http://localhost:31300
```

_Kubernetes_

On minikube

```bash
A8_CONTROLLER_URL=http://$(minikube ip):31200
A8_REGISTRY_URL=http://$(minikube ip):31300
```

On Google Cloud Platform, assign a public IP to the Node running the
controller and the registry, and set the environment variables
respectively.

_IBM Bluemix_

```bash
A8_CONTROLLER_URL=http://<controller route>.mybluemix.net
A8_REGISTRY_URL=http://<registry route>.mybluemix.net
```

where `<controller route>` and `<registry route>` correspond to the routes
that you set for the controller and registry respectively.

### 3. Demo Applications

Two demo applications are provided with this distribution. The first is a
simple `helloworld` application with one microservice. We will use this app
to play with some of Amalgam8's basic version-based routing
capabilities. The second application, called `bookinfo` is a polyglot
application with 4 microservices written using Python, Ruby and Java. We
will use the bookinfo application to demonstrate several features of Amalgam8.


The commands to deploy a sample application for different environments are as
follows:

_Docker Compose_
  
```bash
docker-compose -f <url_to_a8_docker_demos.yaml> up -d
```

_Kubernetes_ on localhost or on Google Cloud

```bash
kubectl create -f <url_to_a8_k8s_demos.yaml>
```

_IBM Bluemix_

1. Create Bluemix routes (DNS names) for the registry, controller and the bookinfo app's gateway:  

   ```bash
   cf create-route <your bluemix space> mybluemix.net -n <your bookinfo route>
   ```

1. Specify the route name in the _.bluemixrc_ in the BOOKINFO_HOSTNAME variable.

1. Deploy the bookinfo application on bluemix.

   ```bash
   ./a8-bluemix create bookinfo
   ```

   Verify that the services are running using the following commands: 

   ```bash
   bluemix ic groups
   ```
