---
layout: page
title: Route Controller
permalink: /docs/controller/
order: 3
---

Amalgam8’s controller enables canary testing, red/black deployments, version-based routing rules, etc., with limited effort. Users can configure how traffic is routed across edge and mid-tier microservices using a variety of user-defined rules. 

The Amalgam8 controller holds user-defined routing and testing rule definitions for the Amalgam8 sidecar. The controller validates and optimizes rule definitions provided by the user via a REST API. Sidecars then obtain and interpret these rule definitions and use information from the Amalgam8 registry to route and manipulate requests.

<!-- TODO: INSERT DIAGRAM -->

## Defining and managing rules <a id="rules"></a>
Rule definitions are managed via the Amalgam8 controller's REST API. A simple commandline interface that calls this REST API is available for basic use cases. The CLI documentation is available [here](linktocli). However, for more advanced functionality or defining rules programmatically, the controller API can be called directly. The following describes basic invocation of the controller REST API using `curl` with the utility `jq` to format the output `json`. For these commands we assume that the controller is running in global authorization mode so that no authentication is required.

There are two basic types of rules,

* routing
* action

### Routing rules
Routing rules define routing behavior. For example:
 
* _Route 75% of requests to ServiceA v1 and the remaining 25% to v2._

Routing rules are useful for canary testing, red/black deployments. The rule described above can be added via a call to the controller REST API:

```bash
curl -X POST <controller URL>/v1/rules -H 'Accept: application/json' --data '{"rules": [{"priority": 1, "route": {"backends": [{"weight": 0.25, "tags": ["v2"]}, {"tags": ["v1"]}]}, "destination": "serviceA"}]}' | jq .
```

Output:

```json
{
  "ids": [
    "b3d6cb84-4676-4bc8-8f84-c9f56f99b63d"
  ]
}
```

The API returns the ID of the newly added rule.

As you can see, the Amalgam8 controller REST API allows for powerful dynamic route definitions. 

### Action rules
Action rules define actions that should be performed in specific circumstances. For instance:

* _Delay all requests from ServiceA v2 to ServiceB v1 by 7 seconds_ 
  
Action rules are useful for testing deployments. The rule described above can be added via a call to the controller REST API:

```bash
curl -i -X POST <controller URL>/v1/rules -H 'Accept: application/json' --data '{"rules": [{"priority": 10, "destination": "ratings", "actions": [{"action": "delay", "duration": 7.0, "probability": 1.0, "tags": ["v1"]}], "match": {"source": {"name": "serviceA", "tags": ["v2"]}}}]}' | jq .
```

Output:

```json
{
  "ids": [
    "74dd54f8-706c-43b2-b229-8c465d19952c"
  ]
}
```

### Listing and deleting rules
To list rules, the following `curl` command can be used:

```bash
curl <controller URL>/v1/rules | jq .
```

The output of the above command should look like this:

```json
{
  "rules": [
    {
      "id": "74dd54f8-706c-43b2-b229-8c465d19952c",
      "priority": 10,
      "destination": "serviceB",
      "match": {
        "source": {
          "name": "serviceA",
          "tags": [
            "v2"
          ]
        }
      },
      "actions": [
        {
          "action": "delay",
          "duration": 7,
          "probability": 1,
          "tags": [
            "v1"
          ]
        }
      ]
    },
    {
      "id": "b3d6cb84-4676-4bc8-8f84-c9f56f99b63d",
      "priority": 1,
      "destination": "serviceA",
      "route": {
        "backends": [
          {
            "weight": 0.25,
            "tags": [
              "v2"
            ]
          },
          {
            "tags": [
              "v1"
            ]
          }
        ]
      }
    }
  ],
  "revision": 2
}
```

Individual rules can be queried by ID or tag.

```bash
curl <controller URL>/v1/rules?id=b3d6cb84-4676-4bc8-8f84-c9f56f99b63d | jq .
```

Output:

```json
{
  "rules": [
    {
      "id": "b3d6cb84-4676-4bc8-8f84-c9f56f99b63d",
      "priority": 1,
      "destination": "serviceA",
      "route": {
        "backends": [
          {
            "weight": 0.25,
            "tags": [
              "v2"
            ]
          },
          {
            "tags": [
              "v1"
            ]
          }
        ]
      }
    }
  ],
  "revision": 2
}
```

Similarly, rules can be removed by ID or tag. 

```bash
curl -i -X DELETE <controllerURL>/v1/rules?id=b3d6cb84-4676-4bc8-8f84-c9f56f99b63d
```

If no query parameters are provided, all the rules are deleted.

```bash
curl -i -X DELETE <controller URL>/v1/rules
```

### Full documentation

Detailed documentation on the route controller's REST API can be found [here](/api/controller/).

Rules passed to the controller are validated against a [JSON schema](http://json-schema.org/) definition. The JSON schema describes the set of possible valid rule definitions. The full JSON schema definition for Amalgam8 controller rules is available [here](/api/controller-rules-schema.json).

## Service references in rules vs services registered in registry
Amalgam8 controller rule definitions reference services. The Amalgam8 registry also keeps track of active services. However, the services referenced in Amalgam8 controller rules and the services registered in the Amalgam8 registry are independent.

* Registry services: are composed of transient endpoints that are registered and maintained by heartbeats from sidecars. Registry endpoints (and therefore the services they are part of) may appear or disappear due to networking, crashes, new deployments, etc.

* Service references in controller rules: are references to a service that may or may not currently exist in registry.

This separation allows routing and other rules to be defined independently of the current state of the environment. Rules are not removed because the service they reference is down. Likewise, rules may reference services that have not yet been registered or no longer exist.
