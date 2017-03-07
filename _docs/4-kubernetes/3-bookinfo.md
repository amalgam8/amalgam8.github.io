---
layout: page
title: Bookinfo
permalink: /docs/kubernetes-integration-bookinfo.html
redirect_from: /docs/kubernetes-integration/bookinfo
category: Kubernetes Integration
order: 4
---

Please refer to [bookinfo](/docs/demo-bookinfo.html) for a detailed description of the
 application and use case.

> Unlike the original bookinfo sample, this walkthrough currently supports fault injection only,
> without validation of failure recovery. These steps may be added in the future.
>
> Before attempting to run this sample, please read through the [prerequisites and caveats](/docs/kubernetes-integration-intro.html#prerequisites-caveats)

## Running the Bookinfo Sample Application

1. Bring up the [control plane](/docs/kubernetes-integration-control-plane.html#deploy)

1. Bring up the application containers:

   ```bash
   $ kubectl create -f examples/k8s-bookinfo.yaml
   ```

   The above command automatically launches the gateway service and the bookinfo application microservices.
   ![Bookinfo app](/docs/figures/amalgam8-example-app-bookinfo.svg)
   Each microservice has a corresponding Kubernetes service associated with it.
   Except for the *reviews* microservice, all other microservices are backed by a single pod.
   The *reviews* microservice has three backing pods, each implementing a different version of the service.

   Confirm that all services and pods are correctly defined and running:

   ```bash
   $ kubectl get services
    NAME          CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
    controller    10.0.0.66    <nodes>       6080:31200/TCP   6m
    details       None         <none>        9080/TCP         17s
    gateway       10.0.0.86    <nodes>       6379:32000/TCP   17s
    kubernetes    10.0.0.1     <none>        443/TCP          7m
    productpage   None         <none>        9080/TCP         17s
    ratings       None         <none>        9080/TCP         16s
    reviews       None         <none>        9080/TCP         16s
   ```

   and

   ```bash
   $ kubectl get pods
    NAME                   READY     STATUS             RESTARTS   AGE
    controller-dzxwk       1/1       Running            0          7m
    details-v1-q6fz0       1/1       Running            0          58s
    gateway-8vdrm          1/1       Running            0          59s
    productpage-v1-mlbm9   2/2       Running            0          59s
    ratings-v1-m5z3b       1/1       Running            0          58s
    reviews-v1-q4brr       0/1       ImagePullBackOff   0          57s
    reviews-v2-kk6v5       2/2       Running            0          57s
    reviews-v3-966g9       2/2       Running            0          57s
   ```


1. Determine the Gateway service URL

   Determine the node on which the `gateway` pod runs, and use the node's IP address as the external gateway IP.

   ```bash
   $ kubectl describe pod gateway-j6vr6 | grep Node
   Node:		node4.k8s.example.com/9.147.221.77
   $ export GATEWAY_URL=9.147.221.77:32000
   ```

### Content Based Routing

1. Set the default routes to v1 of each service

   ```bash
    $ a8ctl rule-create -f examples/bookinfo-default-route-details.yaml
    $ a8ctl rule-create -f examples/bookinfo-default-route-productpage.yaml
    $ a8ctl rule-create -f examples/bookinfo-default-route-ratings.yaml
    $ a8ctl rule-create -f examples/bookinfo-default-route-reviews.yaml
   ```

   You can confirm the routes are defined and valid by inspecting the rules' `Status` field returned in:

   ```bash
   $ kubectl get routingrules -o json
   ```

   Since rule propagation to the proxies is asynchronous, you ahould wait a few seconds for the rules
   to propagate to all pods before attempting to access the application.
   If you open the Bookinfo URL (`http://$GATEWAY_URL/productpage/productpage`) in your browser,
   you should see the bookinfo application `productpage` displayed. Notice that the `productpage`
   is displayed, with no rating stars since `reviews:v1` does not access the ratings service.

1. Route a specific user to `reviews:v2`

   Lets enable the ratings service for test user "jason" by routing productpage traffic to
   `reviews:v2` instances.

   ```bash
   $ a8ctl rule-create -f examples/bookinfo-jason-reviews-v2-route-rules.yaml
   ```

   Confirm the rule is applied and valid:

   ```bash
   $ kubectl get routingrule bookinfo-jason-reviews-v2-route-rules -o json
    {
        "apiVersion": "amalgam8.io/v1",
        "kind": "RoutingRule",
        "metadata": {
            "creationTimestamp": "2017-03-07T19:02:29Z",
            "name": "bookinfo-jason-reviews-v2-route-rules",
            "namespace": "default",
            "resourceVersion": "1593",
            "selfLink": "/apis/amalgam8.io/v1/namespaces/default/routingrules/bookinfo-jason-reviews-v2-route-rules",
            "uid": "9c860db2-0368-11e7-8770-080027c86dc2"
        },
        "spec": {
            "destination": "reviews",
            "id": "bookinfo-jason-reviews-v2-route-rules",
            "match": {
                "headers": {
                    "Cookie": "^(.*?;)?(user=jason)(;.*)?$"
                }
            },
            "priority": 2,
            "route": {
                "backends": [
                    {
                        "tags": [
                            "version=v2"
                        ]
                    }
                ]
            }
        },
        "status": {
            "state": "valid"
        }
    }
   ```

   Log in as user "jason" at the `productpage` web page. You should now see ratings (1-5 stars) next
   to each review.

### Fault Injection

   To test our bookinfo application microservices for resiliency, we will _inject a 7s delay_
   between the reviews:v2 and ratings microservices. Since the _reviews:v2_ service has a
   10s timeout for its calls to the ratings service, we expect the end-to-end flow to
   continue without any errors.

1. Inject the delay

   Create a fault injection rule, to delay traffic coming from user "jason" (our test user).

   ```bash
   $ a8ctl rule-create -f examples/bookinfo-jason-7s-delay.yaml
   ```

   Verify that the rule has been validated:

   ```bash
   $ kubectl get routingrule set-delay-user-jason -o json | sed -n '/status/,/}/p'
      "status": {
         "state": "valid"
      }
   ```

   Allow several seconds to account for rule propagation delay to all pods.

1. Observe application behavior

   If the application's front page was set to correctly handle delays, we expect it
   to load within approximately 7 seconds. To see the web page response times, open the
   *Developer Tools* menu in IE, Chrome or Firefox (typically, key combination _Ctrl+Shift+I_
   or _Alt+Cmd+I_) and reload the `productpage` web page.

   You will see that the webpage loads in about 6 seconds. The reviews section will show
   *Sorry, product reviews are currently unavailable for this book*.

   The reason that the entire reviews service has failed is because our bookinfo application
   has a bug. The timeout between the productpage and reviews service is less (3s) than the
   timeout between the reviews and ratings service (10s). These kinds of bugs can occur in
   typical enterprise applications where different teams develop different microservices
   independently. Amalgam8's fault injection rules help you identify such anomalies without
   impacting end users.

   > Notice that we are restricting the failure impact to user "jason only. If you login
   > as any other user, you would not experience any delays.

### Cleanup

1. Delete the routing rules

   ```bash
   $ a8ctl rule-delete -a -f
   $ kubectl get routingrules    #-- there should be no more routing rules
   No resources found.
   ```

1. Terminate the application pods

   ```bash
   $ kubectl delete -f examples/k8s-bookinfo.yaml
   ```

1. Terminate the [control plane](/docs/kubernetes-integration-control-plane.html#cleanup)
