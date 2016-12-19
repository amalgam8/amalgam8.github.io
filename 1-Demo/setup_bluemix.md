---
layout: page
title: Setup Amalgam8 for Bluemix
permalink: /docs/demo-setup_bluemix.html
redirect_from:
category: Demo
order: 1
---

## Before you begin

Install the [Bluemix CLI](http://clis.ng.bluemix.net/ui/home.html) version 0.4.1
  or later with the IBM-Containers plugin 0.5.800 or later.

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
If you are running any of these tools, please run the following command on those VMs:

```
bash
sudo sysctl -w vm.max_map_count=262144
```

**Note:** You can ssh into the minikube virtual machine by running `minikube ssh`

1. Login to Bluemix and initialize the container environment by running:

   ```
   bluemix login
   bluemix ic init
   ```

1. Create Bluemix routes (DNS names) for the registry, controller and the bookinfo app's gateway:

   ```
   bash
   bluemix cf create-route <your bluemix space> mybluemix.net -n <your registry route>
   bluemix cf create-route <your bluemix space> mybluemix.net -n <your controller route>
   ```

   where `<your bluemix space>` is the name of your Bluemix space and
   `<your route ...>` is a unique route name for `registry` and
   `controller`. For example `my-space-name-a8-registry` and
   `my-space-name-a8-controller`. Make a note of the route names you
   choose for the next step.

1. Customize the _examples/bluemix.cfg_ in the downloaded folder as follows:
    * BLUEMIX_REGISTRY_NAMESPACE should be your Bluemix registry namespace
      obtained from the folowing command:

      ```
      bash
      bluemix ic namespace-get
      ```

    * REGISTRY_HOSTNAME should be the route name assigned to the registry in the previous step

    * CONTROLLER_HOSTNAME should be the route name assigned to the controller in the previous step

1. Deploy the control plane services (registry and controller) on bluemix.

   ```
   bash
   examples/a8-bluemix create controlplane
   ```

   Verify that the controller and registry are running using the following commands:

   ```
   bash
   bluemix ic groups
   ```

   You should see the groups `amalgam8_controller` and `amalgam8_registry` listed in the output.

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
variable. On OS X, run:

```
bash
export PATH=$PATH:${HOME}/Library/Python/2.7/bin
```

On Linux, run:

```
bash
export PATH=$PATH:${HOME}/.local/bin
```

To use the Amalgam8 CLI (`a8ctl`) for the demo walkthroughs we need
to setup two environment variables: `A8_CONTROLLER_URL` and
`A8_REGISTRY_URL`. These variables should point to the publicly accessible
REST endpoints exposed by the Amalgam8 controller and registry respectively.

The value of these variables depends on the environment where you setup the
control plane.

1. Run the following commands:

```
bash
export A8_CONTROLLER_URL=http://<controller route>.mybluemix.net
export A8_REGISTRY_URL=http://<registry route>.mybluemix.net
```

where `<controller route>` and `<registry route>` correspond to the routes
that you set for the controller and registry respectively.

Now that you've configured Amalgam8 for IBM Bluemix, let's move onto running the demo.
