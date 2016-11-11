---
layout: page
title: Bookinfo
permalink: /docs/demo-bookinfo.html
redirect_from: /docs/demo/bookinfo/
category: Demo
order: 3
---

In this demo, we will deploy a simple app that displays information about a
book, similar to a single catalog entry of an online book store. Displayed
on the page is a description of the book, book details (ISBN, number of
pages, and so on), and a few book reviews.

The bookinfo application is broken into four separate microservices:

* *productpage*. The productpage microservice calls the *details* and *reviews* microservices to populate the page.
* *details*. The details microservice contains book information.
* *reviews*. The reviews microservice contains book reviews. It also calls the *ratings* microservice.
* *ratings*. The ratings microservice contains book ranking information that accompanies a book review. 

There are 3 versions of the reviews microservice:

* Version v1 doesn't call the ratings service.
* Version v2 calls the ratings service, and displays each rating as 1 to 5 black stars.
* Version v3 calls the ratings service, and displays each rating as 1 to 5 red stars.

The end-to-end architecture of the application is shown below.

![Bookinfo app](/docs/figures/amalgam8-example-app-bookinfo.svg)

This application is polyglot, i.e., the microservices are written in
different languages. All microservices are packaged with the
[Amalgam8 Sidecar](/docs/sidecar.html) that provides automatic service
registration. For microservices that need to make outbound API
calls, i.e., `productpage` and `reviews`, the sidecar acts as a proxy
routing requests to appropriate instances of upstream services, based on
the rules set forth by the Control Plane.

### Goals of the demo

* We will use Amalgam8's content-based routing feature to selectively
enable `reviews v2` for a specific QA user named `jason`.

* Using Amalgam8's systematic resilience testing feature, we will inject
faults into the communication between microservices and test the failure
recovery capability of the application. The impact of the failure will be
restricted to only the QA user `jason`.

* After fixing the bugs exposed by systematic resilience testing, we will
gradually rollout a new version `reviews v3` similar to a canary rollout.

## Deploy the App

The commands to deploy the bookinfo demo application for different
environments are as follows:

_Docker Compose_
  
```bash
docker-compose -f examples/docker-bookinfo.yaml up -d
```

_Kubernetes_ on localhost or on Google Cloud

```bash
kubectl create -f examples/k8s-bookinfo.yaml
```

_IBM Bluemix_

1. Create Bluemix routes (DNS names) for the registry, controller and the bookinfo app's gateway:  

   ```bash
   bluemix cf create-route <your bluemix space> mybluemix.net -n <your bookinfo route>
   ```

1. Specify the route name in the _examples/bluemix.cfg_ in the BOOKINFO_HOSTNAME variable.

1. Deploy the bookinfo application on bluemix.

   ```bash
   examples/a8-bluemix create bookinfo
   ```

   Verify that the services are running using the following commands: 

   ```bash
   bluemix ic groups
   ```

### List the Services in the App

You can view the microservices that are running using the following command:

```bash
a8ctl service-list
```
    
The expected output is the following:

```bash
+-------------+---------------------+
| Service     | Instances           |
+-------------+---------------------+
| reviews     | v1(1), v2(1), v3(1) |
| details     | v1(1)               |
| ratings     | v1(1)               |
| productpage | v1(1)               |
+-------------+---------------------+
```

There are 4 microservices as described in the diagram above. The `reviews`
microservice has 3 versions v1, v2, and v3. Note that in a realistic
deployment, new versions of a microservice are deployed over time
instead of deploying all versions simultaneously.

## Set the default routes

By default, routes need not be set in Amalgam8. When no routes are present,
Amalgam8 sidecars route requests in a random fashion to one of the
instances of the target microservice. However, _it is advisable to
explicitly set a default route for each microservice, so that when a new
version of a microservice is introduced, traffic to the new version can be
released in a controlled fashion_.

Lets route all of the incoming traffic to version `v1` only for each service.

```bash
a8ctl route-set productpage --default v1
a8ctl route-set ratings --default v1
a8ctl route-set details --default v1
a8ctl route-set reviews --default v1
```

Confirm the routes are set by running the following command:

```bash
a8ctl route-list
```

You should see the following output:

```
+-------------+-----------------+-------------------+
| Service     | Default Version | Version Selectors |
+-------------+-----------------+-------------------+
| ratings     | v1              |                   |
| productpage | v1              |                   |
| details     | v1              |                   |
| reviews     | v1              |                   |
+-------------+-----------------+-------------------+
```

Open
[http://GATEWAY_URL/productpage/productpage](http://localhost:32000/productpage/productpage)
from your browser and you should see the bookinfo application `productpage`
displayed.  Notice that the `productpage` is displayed, with no rating
stars since `reviews:v1` does not access the ratings service.

**Note**: Replace `GATEWAY_HOST_PORT` above with the host and port where
the Amalgam8 gateway is running in your environment (e.g., `localhost:32000`,
`<minikube_ip>:32000`, `bookinfo.mybluemix.net`).  `<minikube_ip>` can 
be found by running `echo $(minikube ip)`

## Content-based routing

Lets enable the ratings service for test user "jason" by routing productpage
traffic to `reviews:v2` instances.

```bash
a8ctl route-set reviews --default v1 --selector 'v2(user="jason")'
```

Confirm the routes are set:

```
a8ctl route-list
```

You should see the following output:

```
+-------------+-----------------+-------------------+
| Service     | Default Version | Version Selectors |
+-------------+-----------------+-------------------+
| ratings     | v1              |                   |
| productpage | v1              |                   |
| details     | v1              |                   |
| reviews     | v1              | v2(user="jason")  |
+-------------+-----------------+-------------------+
```

Log in as user "jason" at the `productpage` web page.
You should now see ratings (1-5 stars) next to each review.

## Systematic Resilience Testing

Instead of taking a Chaos-based approach to resilience testing, Amalgam8
uses a systematic approach to test the resilience of microservices. We
refer to this systematic resilience testing framework as
[Gremlin](https://developer.ibm.com/open/2016/06/06/systematically-resilience-testing-of-microservices-with-gremlin/). Using
the framework with Amalgam8, you can define the set of _faults_ to be
injected into your application and a set of _assertions_ that validate
whether the application has properly recovered from the failure.

In this demo, we will test the bookinfo application with the newly
introduced `reviews:v2` version of the reviews microservice. The
_reviews:v2 service has a 10s timeout for its calls to the ratings 
service. We will _inject a 7s delay_ between the reviews:v2 and ratings
microservices and ensure that the end-to-end flow works without any errors.

### Fault Injection w/ Manual Verification

Lets start with simple fault injection first.

* Lets add a fault injection rule via the `a8ctl` CLI that injects a 7s
  delay in all requests with an HTTP Cookie header containing the value
  `user=jason`. In other words, we are confining the faults only to the QA
  user.

  ```bash
  a8ctl action-add --source reviews:v2 --destination ratings --cookie user=jason --action 'v1(1->delay=7)'
  ```

  Verify the rule has been set by running this command:

  ```bash
  a8ctl action-list
  ```

  You should see the following output:

  ```
  +-------------+----------------+---------------------------+----------+--------------------+--------------------------------------+
  | Destination | Source         | Headers                   | Priority | Actions            | Rule Id                              |
  +-------------+----------------+---------------------------+----------+--------------------+--------------------------------------+
  | ratings     | reviews:v2     | Cookie:.*?user=jason      | 10       | v1(1.0->delay=7.0) | e76d79e6-8b3e-45a7-87e7-674480a92d7c |
  +-------------+----------------+---------------------------+----------+--------------------+--------------------------------------+    
  ```

* Lets see the fault injection in action. Ideally the frontpage of the
  application should take 7+ seconds to load. To see the web page response
  time, open the *Developer Tools* (IE, Chrome or Firefox). The typical key
  combination is (Ctrl+Shift+I) for Windows and (Alt+Cmd+I) in Mac.

  Reload the `productpage` web page.

  You will see that the webpage loads in about 6 seconds. The reviews section
  will show *Sorry, product reviews are currently unavailable for this book*.

_**Impact of fault:**_ If the reviews service has a 10s timeout, the
product page should have returned after 7s with full content. What we saw
however is that the entire reviews section is unavailable.

Notice that we are restricting the failure impact to user `jason` only. If
you login as any other user, say "shriram" or "frank", you would not
experience any delays.

### Fault Injection + Automated Verification

Fault injection alone is insufficient, as it requires manual effort to
monitor the application and identify which services failed to recover. In
any reasonably sized application, this troubleshooting process can get
quite complicated and time consuming.

We'll now use a *gremlin recipe* that describes the application topology
(`topology.json`), reproduces the (7 seconds delay) failure scenario (`gremlins.json`),
and adds a set of assertions (`checklist.json`)
that we expect to pass: each service in the call chain should return `HTTP
200 OK` and the productpage should respond in 7 seconds.

**Note 1:** Set the A8_LOG_SERVER environment variable to point to the
  elasticsearch server created during the control plane setup. By default,
  it points to `localhost:30200`.

**Note 2:** This commands in this section will work only on docker local
  and kubernetes based environments. Automated verification of failure
  recovery currently _does not work for Bluemix deployments_.

* Remove the delay rule that we added in the previous step:

  ```bash
  a8ctl rule-clear
  ```

* Run the recipe using the following command from the main examples folder:

  ```bash
  a8ctl recipe-run --topology examples/bookinfo-topology.json --scenarios examples/bookinfo-gremlins.json --checks examples/bookinfo-checks.json --header 'Cookie' --pattern='user=jason'
  ```

  You should see the following output:

  ```
  Inject test requests with HTTP header Cookie matching the pattern user=jason
  When done, press Enter key to continue to validation phase
  ```

* Inject load into the application. When logged in as user `jason`,
  *reload* the `productpage` web page. Wait a few seconds to let the logs
  propagate from the app/sidecar containers to the logstash server and
  finally to elasticsearch. Then, press Enter on the console where the
  above command was run.

  _This form of manual load injection is for the purpose of demonstration
  only. Ideally, you should use automated load injection tools during this
  phase of testing._

  Expected output:

  ```
  +-----------------------+----------------+----------------+--------+-----------------------------------+
  | AssertionName         | Source         | Destination    | Result | ErrorMsg                          |
  +-----------------------+----------------+----------------+--------+-----------------------------------+
  | bounded_response_time | gateway        | productpage:v1 | PASS   |                                   |
  | http_status           | gateway        | productpage:v1 | PASS   |                                   |
  | http_status           | productpage:v1 | reviews:v2     | FAIL   | unexpected connection termination |
  | http_status           | reviews:v2     | ratings:v1     | PASS   |                                   |
  +-----------------------+----------------+----------------+--------+-----------------------------------+
  ```

  **Note:** When logs from logstash do not appear in elasticsearch by the
  time you hit the Enter key, one or more tests above might fail with the
  error message `No log entries found`. When you encounter this situation,
  re-run the above command (`a8ctl recipe-run ...`) and wait for a longer
  time before hitting the Enter key. We are working on a cleaner fix to
  address the log propagation delay problem.

#### Understanding the output

* We set a 7s delay between `reviews` (v2) and `ratings:v1` service.

* Since the `reviews` service had a 10s timeout, it successfully completed
the API call to the `ratings` service despite the network delay (as shown
in last line of the output above)

* We also saw that the `productpage` service returned a HTTP 200 to the
`gateway` service within the 7s response time limit we had set (the first
two lines in the output above).

While these checks seem to succeed, the webpage output still does not match
our expectation. Why?

_**What went wrong?**_ The answer lies in the failing check between
`productpage` and `reviews` service. The third line, a _failing assertion_
in the above output indicates that the productpage microservice timed out
on its API call to the reviews service (via sidecar) and subsequently
closed the connection. However, the table also shows that the call from
reviews to ratings service was successful.

This behavior suggests that the _productpage service has a smaller timeout
to the reviews service, compared to the timeout duration between the
reviews and ratings service._

What we have here is a typical bug in microservice applications:
**conflicting failure handling policies in different microservices**.  The
systematic resilience testing approach enables you to spot such issues in
production deployments without impacting real users.

#### Fixing the bug

At this point we would normally fix the problem by either increasing the
productpage timeout or decreasing the reviews to ratings service timeout,
terminate and restart the fixed microservice, and then run a gremlin recipe
again to confirm that the productpage returns its response without any
errors.  (Left as an exercise for the reader - change the gremlin recipe to
use a 2.8 second delay and then run it against the v3 version of reviews.)

However, we already have this fix running in v3 of the reviews service, so
we can next demonstrate deployment of a new version.

## Gradually migrate traffic to reviews:v3 for all users

Now that we have tested the reviews service, fixed the bug and deployed a
new version (`reviews:v3`), lets route all user traffic from `reviews:v1`
to `reviews:v3` in a gradual manner.

First, stop any `reviews:v2` traffic:

```bash
a8ctl route-set reviews --default v1
```

Now, transfer traffic from `reviews:v1` to `reviews:v3` with the following series of commands:

```bash
a8ctl traffic-start reviews v3
```

You should see:

```
Transfer starting for reviews: diverting 10% of traffic from v1 to v3
```

Things seem to be going smoothly. Lets increase traffic to reviews:v3 by another 10%.

```bash
a8ctl traffic-step reviews
```

You should see:

```
Transfer step for reviews: diverting 20% of traffic from v1 to v3
```

Lets route 50% of traffic to `reviews:v3`

```bash
a8ctl traffic-step reviews --amount 50
```

We are confident that our Bookinfo app is stable. Lets route 100% of traffic to `reviews:v3`

```bash
a8ctl traffic-step reviews --amount 100
```

You should see:

```
Transfer complete for reviews: sending 100% of traffic to v3
```

If you log in to the `productpage` as any user, you should see book reviews
with *red* colored star ratings for each review.

## Cleanup

To remove the `bookinfo` application,

_Docker Compose_
  
```bash
docker-compose -f examples/docker-bookinfo.yaml kill
```

_Kubernetes_ on localhost or on Google Cloud

```bash
kubectl delete -f examples/k8s-bookinfo.yaml
```

_IBM Bluemix_

```bash
examples/a8-bluemix destroy bookinfo
```
