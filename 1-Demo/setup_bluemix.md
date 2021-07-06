---
layout: page
title: Setup Amalgam8 for Bluemix
permalink: /docs/demo-setup_bluemix.html
redirect_from:
category: Demo
order: 1
---

## Before you begin

Amalgam8 on Bluemix uses the IBM Bluemix Container Service, which requires the Cloud Foundry (cf) and Bluemix (bluemix) plugins. If these are not already installed, then follow these steps.

1. Install the Cloud Foundry (cf) plug-in.
      
        a) Download the latest version of the plug-in from the following GitHub repo: http://github.com/cloudfoundry/cli/releases
        b) Follow the instructions that are available in http://docs.cloudfoundry.org/cf-cli/install-go-cli.html#linux to install the CF plug-in.

2. Install the Bluemix (bluemix) plug-in.

        a) Download the latest version of the plug-in from http://plugins.ng.bluemix.net/ui/home.html.
        b) Extract the file.
        c) Change to the directory `Bluemix_CLI` and install the plug-in. Run the following command as `root`: `./install_bluemix_cli`

     For example, on Ubuntu, the following code installs the Bluemix plugin:

        ```
        # Run the install command as root
          root@bluemix:~# tar -xvf Bluemix_CLI_0.4.2_amd64.tar.gz
          Bluemix_CLI/
          Bluemix_CLI/bx/
          Bluemix_CLI/bx/zsh_autocomplete
          Bluemix_CLI/bx/bash_autocomplete
          Bluemix_CLI/bin/
          Bluemix_CLI/bin/bluemix
          Bluemix_CLI/bin/NOTICE
          Bluemix_CLI/bin/LICENSE
          Bluemix_CLI/bin/bluemix-analytics
          Bluemix_CLI/install_bluemix_cli

          # Change directory
          #root@bluemix:~# cd Bluemix_CLI

          # Install the Bluemix CLI
          root@bluemix:~/Bluemix_CLI# ./install_bluemix_cli
          The Cloud Foundry CLI version 6.22 is already installed.
          Copying files ...
          The Bluemix Command Line Interface (Bluemix CLI) is installed successfully.
          To get started, open a new Linux terminal and enter "bluemix help", or enter "bx help" as short name.
          ```


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

To deploy the control plane in the IBM Bluemix Container Service, complete the following steps.

1. Log in to the Bluemix org and space in which you want to deploy the Amalgam8 control plane, and add the IBM Bluemix Container Service plugin by running the following command:

      ```
      bluemix login
      bluemix plugin repo-add Bluemix https://plugins.ng.bluemix.net
      bluemix plugin install IBM-Containers -r Bluemix
      ```

      You can verify the installation by running ``bluemix plugin list``. The command return resembles the following output:

      ```
      Listing installed plug-ins...
      Plugin Name Version
      IBM-Containers 1.0.0
      ```

2. Start the IBM Bluemix Container Service in the space in which you want to deploy the Amalgam8 control plane by running the following command:

  ```
  bluemix ic init
  ```

  If you are requested to enter a namespace value, be aware that this value is set once for an organization and after it is created it cannot be changed. You might want to check with your organization's administrator before you proceed. Refer to the [IBM Bluemix Container Service](https://console.bluemix.net/docs/containers/container_index.html) documentation for more information.

  For the purposes of running the Amalgam8 demos, you can customize the downloaded _examples/bluemix.cfg_ file to set your environment variables, and then run the a8-bluemix script. After you run the script, skip to [Setting up the Amalgam8 CLI](#a8cli). To learn how to set your environment variables by using the Bluemix CLI, continue following the steps.

3. Collect the information that is required to set values for the Amalgam8 environment variables:

  | Command to set environment variable                          | What it is                                                                                                                                                                                                                                                             | Command to get the value         |
  |--------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------|
  | ``` export ROUTES_DOMAIN=example.bluemix.net ```             | An available domain in your Bluemix org.  In the command, replace `example.bluemix.net` with your chosen Bluemix domain.                                                                                                                                                 | ``` bluemix cf domains ```       |
  | ``` export BLUEMIX_SPACE=example ```                         | The space in your org that you want to deploy Amalgam8 to.  In the command, replace `example` with the name of the space.                                                                                                                                                | ``` bluemix cf target ```        |
  | ``` export BLUEMIX_REGISTRY_NAMESPACE= example ```           | The namespace for your org.  In the command, replace `example` with the namespace.                                                                                                                                                                                       | ``` bluemix ic namespace-get ``` |
  | ``` export BLUEMIX_REGISTRY_HOST=registry.ng.bluemix.net ``` | The IBM Bluemix Container Service registry. The BLUEMIX_REGISTRY_HOST defaults to the container image registry that is associated with the region and organization that your are working on in Bluemix. For example, in US South the value is `registry.ng.bluemix.net`. | ``` bluemix ic info ```          |



4. Run the following commands to create the other required environment variables:

  ```
  export CONTROLLER_IMAGE=a8-controller:latest
  export REGISTRY_IMAGE=a8-registry:latest
  export DOCKERHUB_NAMESPACE=amalgam8
  ```

          The Amalgam8 registry and controller components are deployed to the Bluemix space as container groups. Each container group must be assigned a route so that the components can be accessed. In the environment variables, the route defaults to the name of the Bluemix space that is appended with -a8-registry and -a8-controller. If you want to use different routes, change these environment variables:

  ```
  export REGISTRY_HOSTNAME=$BLUEMIX_SPACE-a8-registry
  export CONTROLLER_HOSTNAME=$BLUEMIX_SPACE-a8-controller
  export REGISTRY_URL=http://$REGISTRY_HOSTNAME.$ROUTES_DOMAIN
  export CONTROLLER_URL=http://$CONTROLLER_HOSTNAME.$ROUTES_DOMAIN
  ```

Your local system is set up for Amalgam8. Next, install the Amalgam8 CLI.



##<a name="a8cli">Setting up the Amalgam8 CLI</a>

The `a8ctl` command line utility provides a convenient way to setup and
manage routes across microservices as well as inspect the state of the
system. The CLI can be installed via the `pip` tool (available as part of
standard Python installation) directly from Python's package repository or
from `a8ctl` [github repository](https://github.com/amalgam8/a8ctl).


1. Run the following command to install the a8ctl into your home directory:

  ```
  bash
  pip install --user a8ctl
  ```

1. Add the location of the `a8ctl` command to the `PATH` environment variable.

  * On OS X, run:

    ```
    bash
    export PATH=$PATH:${HOME}/Library/Python/2.7/bin
    ```

  * On Linux, run:

    ```
    bash
    export PATH=$PATH:${HOME}/.local/bin
    ```

    So that the a8ctl command can communicate with the Amalgam8 control plane, you must set the a8ctl command environment variables for the registry and controller components of the control plane.
    You need to set up two environment variables: `A8_CONTROLLER_URL` and
    `A8_REGISTRY_URL`. These variables should point to the publicly accessible
    REST endpoints exposed by the Amalgam8 controller and registry respectively.

  The value of these variables depends on the environment where you set up the
  control plane.

1. Run the following commands:

  ```
  bash
  export A8_CONTROLLER_URL=$CONTROLLER_URL
  export A8_REGISTRY_URL=$REGISTRY_URL
  ```

Now that you've configured Amalgam8 for IBM Bluemix, let's move onto running the demo.
