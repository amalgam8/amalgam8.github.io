---
layout: page
title: Amalgam8 Demo
permalink: /docs/amalgam8-demo/
order: 1
---

This guide illustrates how to use Amalgam8 for version and content-based
routing for two demo applications. The first is a simple `helloworld`
application with one microservice, while the second is a slighly more
complicated application called `bookinfo` composed of 4 microservices.

The demo can be run on [Docker](https://www.docker.com) and
[Kubernetes](https://kubernetes.io)-based environments locally as well as
on PaaS environments such as [IBM Bluemix](https://bluemix.net) and
[Google Cloud Platform](https://cloud.google.com).

_While Amalgam8 supports multi-tenancy, for the sake of simplicity, this
demo will setup Amalgam8 in single-tenant mode._

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
running Amalgam8

## Deployment

There are two components we need for the demo: the Amalgam8 Control Plane
and the microservice application itself. Lets setup each of these
components one by one.

### Amalgam8 Control Plane

The Amalgam8 Control Plane consists of the
[service registry](/docs/registry/) and the
[route controller](/docs/controller/). The two components are deployed in a
non-HA configuration, with a Redis based datastore backend. In addition to
these two components, for the purposes of this demo, a stock ELK stack is
also included as part of the control plane deployment scripts.

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
   cf create-route <your bluemix space> mybluemix.net -n <your bookinfo route>
   ```

   where `<your bluemix space>` is the name of your Bluemix space and
   `<your route ...>` is a unique route name for `registry`, `controller`
   and `bookinfo`. For example `my-space-name-a8-registry`,
   `my-space-name-a8-controller`, etc. Make a note of the route names you
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

    * BOOKINFO_HOSTNAME should be the route name assigned to the bookinfo gateway in the previous step

1. Deploy the control plane services (registry and controller) on bluemix.

   ```bash
   bluemix/deploy-controlplane.sh
   ```

   Verify that the controller and registry are running using the following commands: 

   ```bash
   bluemix ic groups
   ```
 
   You should see the groups `amalgam8_controller` and `amalgam8_registry` listed in the output.

## Amalgam8 with Docker - local environment <a id="local-docker"></a>

The installation steps below have been tested with the Vagrant sandbox
environment (based on Ubuntu 14.04) as well as with Docker for Mac Beta
(v1.11.2-beta15 or later). These steps have not been tested on Docker for
Windows Beta.

The following instructions assume that you are using the Vagrant
environment. Where appropriate, environment specific instructions are
provided.

1. Download the [Vagrantfile](Vagrantfile) and start the Vagrant environment (or install and setup the equivalent dependencies manually).

   ```bash
   vagrant up
   vagrant ssh
   ```

   Once inside the vagrant box, switch to the Amalgam8 examples folder:

   ```bash
    cd amalgam8/examples
   ```

1. Start the control plane services (service registry, controller) and the ELK stack

   ```bash
   docker/run-controlplane-docker.sh start
   ```


1. The `a8ctl` command line utility can be used to setup routes between microservices. 
   Before we start using the `a8ctl` command line utility, we need to set 
   the `A8_CONTROLLER_URL` and the `A8_REGISTRY_URL` environment variables
   to point to the addresses of the controller and the registry respectively.

    * If you are running the Docker setup using the Vagrant file in the
      `examples` folder or if you are running Docker locally (on Linux or
      Docker for Mac Beta)

      ```bash
      export A8_CONTROLLER_URL=http://localhost:31200
      export A8_REGISTRY_URL=http://localhost:31300
      ```

    * If you are running Docker using the Docker Toolbox with Docker Machine,
      then set the environment variables to the IP address of the VM created
      by Docker Machine. For example, assuming you have only one Docker
      Machine running on your system, the following commands will setup the
      appropriate environment variables:

      ```bash
      export A8_CONTROLLER_URL=`docker-machine ip`
      export A8_REGISTRY_URL=`docker-machine ip`
      ```

1. Confirm everything is working with the following command:

   ```bash
   a8ctl service-list
   ```

   The command shouldn't return any services, since we haven't started any yet.
   It should return the following empty table confirming that the control plane services (and CLI) are working as expected:

   ```
   +---------+-----------+
   | Service | Instances |
   +---------+-----------+
   +---------+-----------+
   ```


1. Deploy the API gateway

   The [API Gateway](http://microservices.io/patterns/apigateway.html) 
   provides a single user-facing entry point for a microservices-based
   application.  You can use the Amalgam8 gateway for different purposes,
   such as version-aware routing, red/black deployments, canary testing, resiliency
   testing, and so on. The Amalgam8 API gateway is a simple lightweight Openresty server (i.e. Nginx)
   that is controlled by the control plane.


   To start the API gateway, run the following command:

   ```bash
   docker-compose -f docker/gateway.yaml up -d
   ```

   Usually, the API gateway is mapped to a DNS route. However, in our local
   standalone environment, you can access it at port 32000 on localhost.
   If you are using Docker directly, then the gateway should be
   accessible at `http://localhost:32000` or `http://dockermachineip:32000`.

1. Confirm that the API gateway is running by accessing
   `http://localhost:32000` from your browser.
   If all is well, you should see a simple **Welcome to nginx!**
   page in your browser.

1. Follow the instructions for the sample that you want to run.

   (a) **helloworld** sample

    * Start the helloworld application:

      ```bash
      docker-compose -f docker/helloworld.yaml up -d
      docker-compose -f docker/helloworld.yaml scale helloworld-v1=2
      docker-compose -f docker/helloworld.yaml scale helloworld-v2=2
      ```
        
    * Follow the instructions for the [Helloworld](apps/helloworld/) example

    * To shutdown the helloworld instances, run the following commands:
   
      ```bash
      docker-compose -f docker/helloworld.yaml kill
      docker-compose -f docker/helloworld.yaml rm -f
      ```

    (b) **bookinfo** sample

    * Start the bookinfo application:
    
      ```bash
      docker-compose -f docker/bookinfo.yaml up -d
      ```

    * Follow the instructions for the [Bookinfo](apps/bookinfo/) example

    * To shutdown the bookinfo instances, run the following commands:
    
      ```
      docker-compose -f docker/bookinfo.yaml kill
      docker-compose -f docker/bookinfo.yaml rm -f
      ```

1. When you are finished, shut down the gateway and control plane services by running the following commands:

   ```
   docker/cleanup.sh
   ```

## Amalgam8 with Kubernetes - local environment <a id="local-k8s"></a>

The following setup has been tested with Kubernetes v1.2.3.

1. Download the [Vagrantfile](Vagrantfile) and start the Vagrant environment (or install and setup the equivalent dependencies manually).

   ```bash
   vagrant up
   vagrant ssh
   ```

   Once inside the vagrant box, switch to the Amalgam8 examples folder and setup the required environment variables

   ```bash
   cd amalgam8/examples
   export A8_CONTROLLER_URL=http://localhost:31200
   export A8_REGISTRY_URL=http://localhost:31300
   ```

   Start Kubernetes, by running the following command:
 
   ```bash
   sudo kubernetes/install-kubernetes.sh
   ```

   **Note:** If you stopped a previous Vagrant VM and restarted it, Kubernetes might be started already, but in a bad state.
   If you have problems, first start by removing previously deployed
   services and uninstalling Kubernetes with the following commands: 
 
   ```bash
   kubernetes/cleanup.sh
   sudo kubernetes/uninstall-kubernetes.sh
   ```

   Alternatively, you can install a local version of Kubernetes via other
   options such as [Minikube](https://github.com/kubernetes/minikube). 
   In this case, make sure to clone the
   [amalgam8 repository](https://github.com/amalgam8/amalgam8) and install
   the Python-based `a8ctl` CLI utility on your host machine.

1. Start the local control plane services (registry and controller) and the ELK stack by running the following script:

   ```bash
   kubernetes/run-controlplane-local-k8s.sh start
   ```

1. Confirm that the control plane is working:

   ```bash
   a8ctl service-list
   ```

   The command shouldn't return any services, since we haven't started any yet.
   It should return the following empty table confirming that the control plane services (and CLI) are working as expected:
    
   ```
   +---------+-----------+
   | Service | Instances |
   +---------+-----------+
   +---------+-----------+
   ```
    
   **Note:** If this did not work, it's probabaly because the image download and/or service initialization took too long.
   This is usually fixed by waiting a minute or two, and then running `kubernetes/run-controlplane-local-k8s.sh stop` and then
   repeating the previous step.
    
   You can also access the registry at `http://localhost:31300` from the host machine
   (outside the Vagrant box), and the controller at `http://localhost:31200` .


1. Run the [API Gateway](http://microservices.io/patterns/apigateway.html) with the following commands:

   ```bash
   kubectl create -f kubernetes/gateway.yaml
   ```
    
   Usually, the API gateway is mapped to a DNS route. However, in our local
   standalone environment, you can access it at port 32000 on localhost.

1. Confirm that the API gateway is running by accessing
    `http://localhost:32000` from your browser. If all is well, you should 
    see a simple **Welcome to nginx!** page in your browser.

1. Visualize your deployment using Weave Scope by accessing
   `http://localhost:30040` . Click on `Pods` tab. You should see a graph of
   pods depicting the connectivity between them. As you create more apps
   and manipulate routing across microservices, the graph changes in
   real-time.
   
1. Follow the instructions for the sample that you want to run.

    (a) **helloworld** sample

    * Start the helloworld application:
    
      ```bash
      kubectl create -f kubernetes/helloworld.yaml
      ```
        
    * Follow the instructions for the [Helloworld](apps/helloworld/) example
 
    * To shutdown the helloworld instances, run the following command:
    
      ```bash
      kubectl delete -f kubernetes/helloworld.yaml
      ```

    (b) **bookinfo** sample

    * Start the bookinfo application:
    
      ```bash
      kubectl create -f kubernetes/bookinfo.yaml
      ```

    * Follow the instructions for the [Bookinfo](apps/bookinfo/) example
    
    * To shutdown the bookinfo instances, run the following command:
    
      ```bash
      kubectl delete -f kubernetes/bookinfo.yaml
      ```

1. When you are finished, shut down the gateway and control plane services by running the following commands:

   ```bash
   kubernetes/cleanup.sh
   ```

## Amalgam8 on IBM Bluemix <a id="bluemix"></a>

To run the [Bookinfo sample app](apps/bookinfo/)
on IBM Bluemix, follow the instructions below. If you are not a Bluemix user, you can register at [bluemix.net](http://bluemix.net/).

1. Download the [Vagrantfile](Vagrantfile) and start the Vagrant environment (or install and setup the equivalent dependencies manually).

   ```bash
   vagrant up
   vagrant ssh
   ```

   Once inside the vagrant box, install the following additional dependencies:
   [CF CLI 6.12.0 or later](https://github.com/cloudfoundry/cli/releases),
   [Bluemix CLI 0.3.3 or later](https://clis.ng.bluemix.net/),

1. Switch to the Amalgam8 examples folder

   ```bash
   cd amalgam8/examples
   ```

1. Login to Bluemix and initialize the container environment using 

   ```
   bluemix login
   bluemix ic init
   ```

1. Create Bluemix routes (DNS names) for the registry, controller and the bookinfo app's gateway:  

   ```bash
   cf create-route <your bluemix space> mybluemix.net -n <your registry route>
   cf create-route <your bluemix space> mybluemix.net -n <your controller route>
   cf create-route <your bluemix space> mybluemix.net -n <your bookinfo route>
   ```

   where `<your bluemix space>` is the name of your Bluemix space and
   `<your route ...>` is a unique route name for `registry`, `controller`
   and `bookinfo`. For example `my-space-name-a8-registry`,
   `my-space-name-a8-controller`, etc. Make a note of the route names you
   choose for the next step.


1. Customize the `amalgam8/examples/bluemix/.bluemixrc` file as follows:
    * BLUEMIX_REGISTRY_NAMESPACE should be your Bluemix registry namespace, e.g. ```bluemix ic namespace-get```
    * BLUEMIX_REGISTRY_HOST should be the Bluemix registry hostname. This needs to be set only if you're targeting a Bluemix region other than US-South.
    * REGISTRY_HOSTNAME should be the route name assigned to the registry in the previous step
    * CONTROLLER_HOSTNAME should be the route name assigned to the controller in the previous step
    * BOOKINFO_HOSTNAME should be the route name assigned to the bookinfo gateway in the previous step
    * ROUTES_DOMAIN should be the domain used for the Bluemix routes (e.g., mybluemix.net)

1. Deploy the control plane services (registry and controller) on bluemix.

   ```bash
   bluemix/deploy-controlplane.sh
   ```

   Verify that the controller and registry are running using the following commands: 

   ```bash
   bluemix ic groups
   ```
 
   You should see the groups `amalgam8_controller` and `amalgam8_registry` listed in the output.

1. Configure the Amalgam8 CLI according to the routes defined in `.bluemixrc`. For example

   ```
   export A8_CONTROLLER_URL=http://mya8-controller.mybluemix.net
   export A8_REGISTRY_URL=http://mya8-registry.mybluemix.net
   ```

1. Run the following command to confirm the control plane is working:

   ```bash
   a8ctl service-list
   ```

   The command shouldn't return any services, since we haven't started any yet.
   It should return the following empty table confirming that the control plane services (and CLI) are working as expected:
    
   ```
   +---------+-----------+
   | Service | Instances |
   +---------+-----------+
   +---------+-----------+
   ```

1. Deploy the API Gateway and the Bookinfo app.

   ```bash
   bluemix/deploy-bookinfo.sh
   ```

   Follow the instructions for the [Bookinfo](apps/bookinfo/) example

   **Note 1:** When you reach the part where the tutorial instructs you to open, in your browser, the bookinfo application at
    `http://localhost:32000/productpage/productpage`, make sure to change
    `http://localhost:32000` to `http://${BOOKINFO_HOSTNAME}.mybluemix.net`
    (substitute BOOKINFO_HOSTNAME with the value defined in the
    `.bluemixrc` file).

    **Note 2:** The Bluemix version of the bookinfo sample app does not yet
    support running the Gremlin recipe. We are working on integrating the
    app with the Bluemix Logmet services (ELK stack), to enable support for
    running Gremlin recipes.

1. When you are finished, shut down the gateway and control plane servers by running the following commands:

   ```bash
   bluemix/kill-bookinfo.sh
   bluemix/kill-controlplane.sh
   ```

## Amalgam8 on Google Cloud Platform <a id="gcp"></a>

1. Setup [Google Cloud SDK](https://cloud.google.com/sdk/) on your machine

1. Setup a cluster of 3 nodes

1. Launch the control plane services

   ```bash
   kubernetes/run-controlplane-gcp.sh start
   ```

1. Locate the node where the controller and registry are running and assign an external IP to the node.

1. Deploy the API gateway

   ```bash
   kubectl create -f kubernetes/gateway.yaml
   ```

   Obtain the public IP of the node where the gateway is running. This will
   be the be IP at which the sample app will be accessible.

1. Visualizing your deployment with Weave Scope

   ```bash
   kubectl create -f 'https://scope.weave.works/launch/k8s/weavescope.yaml' --validate=false
   ```

   Once weavescope is up and running, you can view the weavescope dashboard
   on your local host using the following commands
  
   ```bash
   kubectl port-forward $(kubectl get pod --selector=weavescope-component=weavescope-app -o jsonpath={.items..metadata.name}) 4040
   ```
  
   Open `http://localhost:4040` on your browser to access the Scope UI. Click on `Pods` tab. 
   You should see a graph of pods depicting the connectivity between them. As you create 
   more apps and manipulate routing across microservices, the graph changes in real-time.

1. You can now deploy the sample apps as described in
   [Deploying sample apps](#local-k8s-samples) section under the
   [local Kubernetes installation](#local-k8s) instructions. Remember to
   replace the IP address `localhost` with the public IP address of the
   node where the gateway service is running on the Google Cloud Platform.
