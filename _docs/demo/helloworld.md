---
layout: page
title: Hello World!
permalink: /docs/demo/helloworld/
category: Demo
order: 2
---

This demo starts two versions of a helloworld microservice, to demonstrate
how Amalgam8 can be used to split incoming traffic between the two
versions. You can define the proportion of traffic to each microservice as
a percentage.

## Deploy the App

The commands to deploy the helloworld demo application for different
environments are as follows:

_Docker Compose_
  
```bash
docker-compose -f examples/docker-helloworld.yaml up -d
```

_Kubernetes_ on localhost or on Google Cloud

```bash
kubectl create -f examples/k8s-helloworld.yaml
```

_IBM Bluemix_

1. Create Bluemix routes (DNS names) for the registry, controller and the helloworld app's gateway:  

   ```bash
   cf create-route <your bluemix space> mybluemix.net -n <your helloworld route>
   ```

1. Specify the route name in the _.bluemixrc_ in the HELLOWORLD_HOSTNAME variable.

1. Deploy the helloworld application on bluemix.

   ```bash
   ./a8-bluemix create helloworld
   ```

   Verify that the services are running using the following commands: 

   ```bash
   bluemix ic groups
   ```

### Listing the Services in the App

You can view the microservices that are running using the following command:

```bash
a8ctl service-list
```
    
The expected output is the following:

```bash
+------------+--------------+
| Service    | Instances    |
+------------+--------------+
| helloworld | v1(2), v2(2) |
+------------+--------------+
```

There are 4 instances of the helloworld service. Two are instances of
version "v1" and the other two belong to version "v2".

## Version-based routing

1. Lets send all traffic to the v1 version of helloworld. Run the following command:

   ```bash
   a8ctl route-set helloworld --default v1
   ```

1. We can confirm the routes are set by running the following command:

   ```bash
   a8ctl route-list
   ```

   You should see the following output:

   ```bash
   +------------+-----------------+-------------------+
   | Service    | Default Version | Version Selectors |
   +------------+-----------------+-------------------+
   | helloworld | v1              |                   |
   +------------+-----------------+-------------------+
   ```

1. Confirm that all traffic is being directed to the v1 instance, by running the following cURL command multiple times:

   ```bash
   curl http://GATEWAY_HOST_PORT/helloworld/hello
   ```

   **Note**: Replace `GATEWAY_HOST_PORT` above with the host and port where
   the Amalgam8 gateway is running in your environment (e.g., `localhost:32000`,
   `<minikube_ip>:32000`, `helloworld.mybluemix.net`)

   You can see that the traffic is continually routed between the v1 instances only, in a random fashion:

   ```bash
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v1, container: helloworld-v1-p8909
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v1, container: helloworld-v1-qwpex
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v1, container: helloworld-v1-p8909
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v1, container: helloworld-v1-qwpex
   ...
   ```

1. Next, we will split traffic between helloworld v1 and v2

   Run the following command to send 25% of the traffic to helloworld v2, leaving the rest (75%) on v1:
    
   ```bash
   a8ctl route-set helloworld --default v1 --selector 'v2(weight=0.25)'
   ```

1. Run this cURL command several times:

   ```bash
   curl http://GATEWAY_URL/helloworld/hello
   ```

   You will see alternating responses from all 4 helloworld instances, where approximately 1 out of every 4 (25%) responses
   will be from a "v2" instances, and the other responses from the "v1" instances:

   ```bash
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v1, container: helloworld-v1-p8909
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v1, container: helloworld-v1-qwpex
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v2, container: helloworld-v2-ggkvd
   $ curl http://localhost:32000/helloworld/hello
   Hello version: v1, container: helloworld-v1-p8909
   ...
   ```

   Note: if you use a browser instead of cURL to access the service and continually refresh the page, 
   it will always return the same version (v1 or v2), because a cookie is set to maintain version affinity.
   However, the browser still alternates in a random manner between instances of the specific version.
